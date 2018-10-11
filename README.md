# Docker-Use
docker 使用心得


### Homebrew 安装docker
    * brew cask install docker
###  Docker 安装部署RabbitMQ
    *  docker search rabbitmq:management   management web管理界面
    *  docker pull rabbitmq:management
    *  docker run -d -p 5672:5672 -p 15672:15672 --name rabbitmq rabbitmq:management
    *  访问管理界面的地址就是 http://[宿主机IP]:15672，可以使用默认的账户登录，用户名和密码都guest

### Docker 安装部署mysql
    * 使用mysql镜像创建或启动MySQL容器时，可以先将镜像下载到本地: docker pull mysql 
    * 也可以直接使用以下命令来启动MySQL实例： 
    * $ docker run -p 3306:3306 --name itbilu-mysql -e MYSQL_ROOT_PASSWORD=my-pass -d mysql:5.7
    * 这样，我们就创建了一个名为itbilu-mysql的MySQL数据库服务器容器实例。在创建数据库时，通. 过环境变量MYSQL_ROOT_PASSWORD设置数据库的root密码，还通过5.7标签指定了所使用的镜像版本

### Spring boot 项目生成docker 镜像 并运行
    * 第一步 pom 文件添加docker 插件配置
```pom
<plugin>
                <groupId>com.spotify</groupId>
                <artifactId>docker-maven-plugin</artifactId>
                <version>0.4.3</version>
                <configuration>
                    <imageName>gntdcserver</imageName>
                    <dockerDirectory>${project.build.directory}</dockerDirectory>
                    <resources>
                        <resource>
                            <targetPath>/</targetPath>
                            <directory>${project.build.directory}</directory>
                            <include>${project.build.finalName}.jar</include>
                        </resource>
                    </resources>
                </configuration>
            </plugin>
```
    * 第二步 新建一个Dockerfile 文件 
```docker
FROM openjdk:8-jdk-alpine  // jar运行依赖的镜像 
VOLUME /tmp // 生成临时文件夹
EXPOSE 8089  //对外的端口
ADD target/mycar-0.0.1-SNAPSHOT.jar mycar.jar // 改名字
ENTRYPOINT ["java","-jar","/mycar.jar"]
```
     * 第三步 docker build [Dockerfile所在目录]
```file
docker build -t mycar /Users/beifeitu/developer/java/mycar
```
    * 第四步 运行镜像
``` run 
docker run -d -p 8089:8089 mycar
```
