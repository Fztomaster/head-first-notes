# Record Common Bugs
## 1.fileupload插件调用upload.parseRequest(request)解析得到空值问题
在springmvc.xml中可能配置了如下的代码
```xml
<!--配置文件解析器对象，要求id必须是multipartResolver-->
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <property name="maxUploadSize" value="2097152"></property>
</bean>
```
## 2.tomcat的war和war exploded的问题
* **项目工程名:war**这个tomcat部署打包之后会把web项目放在tomcat的webapps目录下
* **项目工程名:war exploded**这个部署打包后放在当前maven项目目录的target目录下
## 3.springmvc跨服务器上传图片报错400
错误提示：
```text
com.sun.jersey.api.client.UniformInterfaceException: PUT http://localhost:9090/fileuploadserver/upload/4D3A90BC0AB649E3910EF4F6ECEA50DC_重邮5.jpg returned a response status of 400 Bad Request
```
原因：上传中文名称的图片，没有转码
解决方案：
```java
// 处理上传中文名称的图片，returned a response status of 400 Bad Request
filename = URLEncoder.encode(filename, "utf-8");
```
## 4.springmvc跨服务器上传图片报错409
原因：保存图片的服务器没有upload文件夹
解决方案：在tomcat的webapps下的图片服务器项目目录里创建一个upload文件夹