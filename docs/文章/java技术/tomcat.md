## Tomcat 支持中文路径

https://blog.csdn.net/MAO_JIN_DAO/article/details/78485044

在/conf/server.xml文件<Connector ports=”8080” ……/>中添加

URIEncoding=“UTF-8” useBodyEncodingForURI=”true”

然后重启tomcat。