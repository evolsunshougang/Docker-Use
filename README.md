# Docker-Use
docker 使用心得

### Homebrew 安装docker（mac）
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
    docker run -p 3306:3306 --restart always --name mymysql -v /etc/localtime:/etc/timezone:rw -v /etc/localtime:/etc/localtime:rw -e MYSQL_ROOT_PASSWORD=asdf@#123 -d  mysql:5.7 --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
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
### Docker 安装 GitLab
    * 下载镜像
 ```pull 
 sudo docker pull gitlab/gitlab-ce:latest
 ```
    * 启动镜像
        * hostname指定容器中绑定的域名
            * 会在创建镜像仓库的时候使用到
            * 这里绑定  192.168.253.10
        * publish 端口映射
            * 冒号前面是宿主机端口，后面是容器expose出的端口
            * https 端口443
            * http 端口80
            * ssh 端口22
        * volume
            * volume 映射，冒号前面是宿主机的一个文件路径
            * 后面是容器中的文件路径
             
| 当地的位置 | <span class="Apple-tab-span" style="white-space:pre"></span>集装箱位置 | <span class="Apple-tab-span" style="white-space:pre"></span>用法 |
| --- | --- | --- |
| /mnt/gitlab/data | /var/opt/gitlab | 用于存储应用数据 |
| /mnt/gitlab/logs | /var/log/gitlab  | 用于存储日志  |
| /mnt/gitlab/config | /etc/gitlab | 用于存储GitLab配置文件
            
``` run 
sudo docker run --detach \
--hostname 192.168.253.10 \
--publish 8443:443 --publish 8880:80 --publish 2222:22 \
--name gitlab \
--restart always \
--volume /mnt/gitlab/config:/etc/gitlab \
--volume /mnt/gitlab/logs:/var/log/gitlab \
--volume /mnt/gitlab/data:/var/opt/gitlab \
gitlab/gitlab-ce:latest
```
    * ssh 登录配置 
``` config
    /etc/gitlab/gitlab.rb
    gitlab_rails['gitlab_shell_ssh_port'] = 2289
```
    * 邮件配置（因为GitLab Docker映像没有安装SMTP服务器）  
``` smtp
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.server"
gitlab_rails['smtp_port'] = 465
gitlab_rails['smtp_user_name'] = "smtp user"
gitlab_rails['smtp_password'] = "smtp password"
gitlab_rails['smtp_domain'] = "example.com"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_openssl_verify_mode'] = 'peer'

# If your SMTP server does not like the default 'From: gitlab@localhost' you
# can change the 'From' with this setting.
gitlab_rails['gitlab_email_from'] = 'gitlab@example.com'
gitlab_rails['gitlab_email_reply_to'] = 'noreply@example.com'
```
    * 默认情况下，为SMTP启用SSL。如果您的SMTP服务器不支持通过SSL进行通信，请使用以下设置：
```
gitlab_rails['smtp_enable'] = true;
gitlab_rails['smtp_address'] = 'localhost';
gitlab_rails['smtp_port'] = 25;
gitlab_rails['smtp_domain'] = 'localhost';
gitlab_rails['smtp_tls'] = false;
gitlab_rails['smtp_openssl_verify_mode'] = 'none'
gitlab_rails['smtp_enable_starttls_auto'] = false
gitlab_rails['smtp_ssl'] = false
gitlab_rails['smtp_force_ssl'] = false
```
    * GitLab shell 命令
        * 重启gitlab服务
            * gitlab-ctl restart
        * 查看gitlab运行状态
            * gitlab-ctl status
        * 停止gitlab服务
            * gitlab-ctl stop
        * 查看gitlab运行日志
            * gitlab-ctl tail
        * 停止相关数据连接服务
            * gitlab-ctl stop unicorn
            * gitlab-ctl stop sideki
```shell
sudo docker exec -it gitlab /bin/bash
``` 
    * 修改ssh端口 
        * 修改/mnt/gitlab/gitlab.rb [gitlab.yml中的配置会被这个给覆盖]
    * 在后面修改自己的ssh端口
        * gitlab_rails['gitlab_shell_ssh_port'] = 21386
    * gitlab-ctl reconfigure
        * 默认是22端口，直接访问则不会出现端口的。）
       * sudo gitlab-ctl restart
       * 修改http路径
           * /mnt/gitlab/data/gitlat-rails/gitlab.yml
