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
