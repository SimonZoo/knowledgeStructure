## 通过Power BI REST API 导出图表文件

### 1. 获得 access token 

可以以账号或者服务的形式来获取token，这里主要说一个通过服务获取token。

1. 在azure中找到“应用注册”，添加新应用，注册完成后获得一个服务主体
2. 授予注册的本地应用程序相关 Windows Active Directory 和 Power BI Service 权限
3. 添加权限后需点击“授予管理员同意”
4. 在 Azure Active Directory 中新建一个安全组，并将注册应用生成的服务主体加到成员中
5. 登录Power BI Service 网站授予相关admin权限
6. 选择 “管理门户” 进行相关授权，将创建的安全组添加到”允许主体服务访问 Power BI API”
7. 在工作区访问中，将服务主体设置为此工作区的管理员
8. 拿到租户ID、client id（在azure概述中），创建一个client secret（azure左边菜单栏证书和密码中）



前期配置结束，对使用过Azure和Power BI的用户来说上面这块应该比较简单，或者直接发邮件/提工单给服务商，会有完整的流程图。

有了租户ID和设置完权限之后，通过POST请求token

```http t
POST https://login.chinacloudapi.cn/{租户ID}/oauth2/token

Content-Type:application/x-www-form-urlencoded

```



Body 写成JSON清晰一点

```JSON
{

    "grant_type":"client_credentials",

    "resource":"https://analysis.chinacloudapi.cn/powerbi/api",

    "client_id":"注册应用的应用程序ID",

    "client_secret":"应用程序密码"

}
```



在POSTAMAN返回数据中看到access token，就是接下来要用到token。



## 2. 获得要导出报表的 export id

**注意点**

1. Power BI 的REST API 文档的API给出的示例都是.com，中国区使用REST API的话，都要改成.cn
2. 根据报表类型（是否属于group）选择对应的API
3. 导出文件类型必选
4. 导出部分文件例如Excel，只有分页报表才能导出，普通报表不支持。分页报表在开通Power BI Premium功能后才能使用，开通后，普通报表无法转换为分页报表，需要重新编辑制作分页报表
5. 通过API导出文件，可以具体到某一个报表页，不能精确到报表页中的每一个表格；要想精确到每一个表格，可以用power BI JavaScript client中的 `export data` 方法导出csv文件。
6. 带上token，token_type 为 Bearer

```http
POST https://api.powerbi.cn/v1.0/myorg/groups/{groupID}/reports/reportID/ExportTo
```



body：

```json
{
    "format": "PPTX"
}
```



返回的数据中带一个id，该id为`export id`



### 3. 查询导出进度

```http
GET https://api.powerbi.cn/v1.0/myorg/groups/{groupID}/reports/{reportID}/exports/{exportID}
```

返回数据



```json
{
  	...,
    "status": "Running", // 表示文件处理中，当处理完成后，状态变为 Succeeded
    "percentComplete": 71, // 表示进度，处理完成后为 100 
  	"resourceLocation": "download link" // 处理完成后返回的数据中会有这个字段，可以通过该链接下载
	  ...
}
```



## 4. 下载文件

```http
GET https://api.powerbi.cn/v1.0/myorg/groups/{groupID}/reports/{reportID}/exports/{exportID}/file
```

这个API就和`resourceLocation`返回的地址一样，在POSTMAN直接将返回保存为文件，就是导出的对应文件。



前端下载的话，可以通过XMLHttPResponse，设置返回数据为blob，创建对应的URL，用`<a>`标签的`download`s属性实现下载。