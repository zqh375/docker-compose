# docker-compose
* 基于docker快速一键构建php开发环境，如有需要添加扩展，请前往php目录下的dockerfile文件进行添加。

# 使用的版本
1. Nginx 1.16
2. PHP 8.2-fpm-alpine
3. Mysql 8.0
4. redis 4.2.0
5. swoole 5.1.10
6. mongodb 1.8.2
7. rabbitmq 1.10.2
8. git 2.26.2
9. composer 1.10.9
10. protobuf

# 构建前请配置docker镜像加速 https://www.runoob.com/docker/docker-mirror-acceleration.html

# 使用方法 1
1. 将项目代码git clone到任意目录，进入到目录执行 docker-compose up -d
2. 命令说明：第一次使用的话会根据目录下的docker-compose.yml进行镜像构建，构建完毕后所有容器会在后台运行，相当于docker-composer build && docker-composer start - d 。mac/linux 直接执行 bash build.sh 
3. 其中php的镜像我已经构建好放到docker hub上了，拉取的时候网速差的时候会稍微耗时一些。

# 使用方法 2
1. 打开目录下的docker-compose.yml,将php服务下的build、context和image注释去掉，并把image: ****注释掉。
2. 执行docker-compose up -d 进行镜像构建和容器构建。
3. 方法2支持自定义php镜像，如若要新增php扩展请前往php目录下修改dockerfile文件。建议在文件最下面另起一层RUN进行添加，这样做的目的使其前面几层的RUN缓存不会丢失从而引起重新下载扩展包构建非常耗时。

* **提示：构建镜像过程中下载php扩展包可能因为网络问题失败，请换个网络并再次尝试**。

* **如果构建成功，将会在控制台看到以下内容**：
* Composer (version 1.10.9) successfully installed to: /var/www/html/composer.phar
* Use it: php composer.phar
* Removing intermediate container 6a467dce4398
*  ---> cf9d9d21ca0f
* Step 6/6 : CMD ["php-fpm"]
*  ---> Running in 5c2472eae7ae
* Removing intermediate container 5c2472eae7ae
*  ---> 5a623ebb35a3
* Successfully built 5a623ebb35a3
* Successfully tagged php-fpm:7.3

-------

#  **输入docker images**:
| REPOSITORY         | TAG         | IMAGE ID     | CREATED      | SIZE   |
|--------------------|-------------|--------------|--------------|--------|
| guqingfeng/babybus | v1          | 95d0e12cda40 | 10 hours ago | 414MB  |
| mysql              | 8.0         | e3fcc9e1cc04 | 7 days ago   | 544MB  |
| redis              | 5.0-alpine  | 893bb508e38f | 7 days ago   | 30.3MB |
| nginx              | 1.16-alpine | 5fad07aba15a | 6 months ago | 21.8MB |
| consul             | latest      | 2823bc69f80f | 4 weeks ago  | 120MB  |
| openzipkin/zipkin  | latest      | 9b4acc3eb019 | 3 weeks ago  | 150MB  |


# 输入docker ps 显示四个容器则表示构建成功

| CONTAINER ID | IMAGE                 | COMMAND                | CREATED       | STATUS       | PORTS                            | NAMES        |
|--------------|-----------------------|------------------------|---------------|--------------|----------------------------------|--------------|
| 96b7619389cb | nginx:1.16-alpine     | "nginx -g 'daemon of…" | 4 minutes ago | Up 4 minutes | 0.0.0.0:80->80/tcp               | nginx        |
| 53032256842e | mysql:8.0             | "docker-entrypoint.s…" | 4 minutes ago | Up 4 minutes | 0.0.0.0:3306->3306/tcp，33060/tcp | mysql        |
| c4b1285cf2f6 | guqingfeng/babybus:v1 | "docker-php-entrypoi…" | 4 minutes ago | Up 4 minutes | 9000/tcp                         | php          |
| 44c65d4d79c0 | redis:5.0-alpine      | "docker-entrypoint.s…" | 4 minutes ago | Up 4 minutes | 0.0.0.0:6379->6379/tcp           | redis-server |
| d4bc5e0b029f | consul:latest         | "docker-entrypoint.s…" | 4 minutes ago | Up 4 minutes | 8300-8302/tcp, 8301-8302/udp, 8600/tcp, 8600/udp, 0.0.0.0:8500->8500/tcp           | consul |
| 61d1513a8564 | openzipkin/zipkin     | "start-zipkin"         | 4 minutes ago | Up 4 minutes | 9410/tcp, 0.0.0.0:9411->9411/tcp  | zipkin |


# 注意事项与备注
* docker-composer 构建的网络默认以docker-compose.yml上级文件名为名称，输入docker network ls 可以看到名字为docker-compose_local-net的bridge类型网络。
* 添加新的php扩展时请重新执行docker-compose build进行镜像构建和容器重启。
* **php映射在docker中的工作目录为当前目录下的www**。你也可以在docker-composer.yml进行修改并重新build和restart容器。
* 普通php项目nginx配置可以参考nginx/conf.d/my.admin.conf 
* swoole项目nginx配置可以参考nginx/conf.d/my.dockerapi.conf
* 记得添加本机host 如 127.0.0.1 my.admin.com 
* mysql8.0连接失败(1045错误)，进入mysql容器内，输入 mysql -h localhost -u root -p 紧接着输入密码123456进入mysql。执行use mysql;
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'root';
* **如若不是添加php扩展，请勿随意修改dockerfile文件，即使添加注释也会使file cache失效导致php镜像重新build**。
* 本地调试php连接Mysql或者redis时不再填写127.0.0.1，而是应该填写通过docker inspect 容器id | grep IPAddress 查找出来的容器ip。
* 执行composer install 时，可能会出现“root用户不能使用的情况”，需要执行：composer install --no-plugins --no-scripts
* composer install后还是找不到类的话，请尝试composer dump-autoload 更新命名空间。


# docker-composer 命令介绍
* 启动容器：docker-compose up -d 或者 docker-compose start
* 关闭容器：docker-compose stop
* 关闭并删除这几个docker容器：docker-compose down
* docker-compose的其他命令请查看其官方文档

# docker 命令介绍
* 查看所有镜像：docker images
* 查看所有容器：docker ps 或者docker ps -a
* 查看容器日志：docker logs 容器id
* 进入容器：docker exec -it 容器id或容器名称 /bin/bash
* 退出容器：exit
* 删除某个镜像：docker rmi 镜像id ps:需要停止删除由它生产的容器才能成功删除
* 停止某个容器：docker stop 容器id
* 删除某个容器：docker rm 容器id
* 停止和删除所有容器：docker stop $(docker ps -q) & docker rm $(docker ps -aq)

# 更新项目源重新搭建步骤
1. git pull项目代码拉取最新的docker-compose.yml
2. 执行 docker-compose down 删除构建好的容器。
3. 执行 clear.sh 或者 docker stop $(docker ps -q) & docker rm $(docker ps -aq) 停止和删除所有容器
4. 执行 start.sh 或者 重新执行docker-compose up -d。

# 举例
1. 在docker-compose目录下执行sh start.sh 启动所有容器。
2. 在docker-compose目录下执行sh php.sh 进入php容器。
3. 在hyperf目录中执行php bin/hyperf.php start 启动hyperf。
