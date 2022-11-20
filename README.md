# LearnNotes

## Nginx/Tomcat部署前后分离项目
    1、几个服务器节点
    2、两台或多台服务部署后端
    3、拉代码：git clone "项目地址"

### 前端部署
    1、将前端项目打包，上传到前端服务器中你定好的目录下（最好是打包，因为直接整包上传可能会丢隐藏文件）；
    2、上传成功，cd到项目所在目录，解压前端项目：unzip 项目文件.zip
    3、cd 项目名：
      a.安装依赖：npm install --unsafe-perm --registry=https://registry.npm.tobao.org
      b.安装完成后进行打包：npm run build:prod
      c.打包完成后，会在项目中生成一个“dist"文件夹
    
    ### 部署
       1、进入服务器是nginx目录，修改nginx/conf目录下：nginx.conf文件
          user root; // 如果项目在root下，需要打开这行代码，避免访问权限问题
          ……
           server {
              ……
              location / {
                root  /项目地址/dist;
                index index.html index.htm;
              }
              ……
           }
        
        2、进入nginx所在目录，cd sbin/，直接输入 ./nginx 启动前端项目。
           

### 后端部署
    1、部署前，用IDEA开发工具打开后台项目，修改配置（application.yml）：
        a.数据库配置（改成数据库所在的服务器地址）
        b.Redis地址修改（实际配置的主机地址）
    2、修改logback.xml
        a.修改控制台 encode => UTF-8
        b.修改日志的路径配置
    3、基础配置修改后，将后端项目上传到需要部署的服务器定好的目录下;
    4、如果是Spring Boot项目，它内嵌Tomcat，打jar包比较好；
    5、进入项目：，cd 项目,输入 mvn package 进行项目打包，项目目录下会生成一个target文件夹；
    6、如果有多台服务器，在对应的服务器重复上面 3-5步骤，也可以直接打之前打过的包，复制到其他服务器中；
    7、进行jar包所在目录，执行 nohup java -jar 项目名.jar & （后台执行）
    
    如果要打war的话：
    1、用IDEA打开项目，修改pom.xml：
        <packaging>war<packaging>
    2、将项目的内嵌的Tomcat去掉
        <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-tomcat</artifactId>
          <scope>provided</scope>
        </dependency>
     3、新加一个启动类：继承 SpringBootServletInitializer，重写configure方法，指向builder.sources(原来的启动类名.class);
     4、将修改后的项目上传到服务器定好的目录下，进行打包：mvn package（如果该服务器之前已经打过一次包，在打包前mvn clean：将target清理掉）；
     5、将打包后的 war 包放入服务器安装的Tomcat/webapps/目录下，修改tomcat/conf目录下server.xml文件，将项目设置为根目录访问；
        在<Host>下添加一个配置：
        <Context path="/" docBase="tomcat下项目所在地址" reloadable="false" />
      6、输入命令 cd webapps/，执行命令：service tomcat start。
   
## 前端页面后台请求问题
    在项目第一次部署时，可能会出面404等请求问题，这里需要F12，找到Network，按F5刷新，看一下Header->Request URL下的请求路径是否在nginx.conf文件中配置；
    如果没有需要打开nginx.conf文件进行配置：
      location /prod-api/ {
        proxy_set_hear Host $http_host;
        proxy_set_hear X-Real_IP $remote_addr;
        proxy_set_hear REMOTE-HOST $remote_addr;
        proxy_set_hear X-Forwarded-For $proxy_add_x_farwarded_for;
        proxy_pass http://服务器IP:端口号/;
      }
      
      如果是多节点部署后台，还需要在nignx.conf中添加配置：
      1、upstream 自己起个名{
        server 服务器1的IP:端口号 weight=5; // weight权重
        server 服务器2的IP:端口号 weigth=4
      }
      2、修改 proxy_pass http://自己起个名/
      
      
### 看日志
    1、cd /修改的日志目录
    2、tail -f sys-info.log
    
    
    
    
    
