---
layout: posts
title: Linux搭建Jenkins持续化集成部署项目
date: 2021-04-13 15:54:47
categories: 学习笔记
tags: [Jenkins, 持续化集成CI/CD]
---



> 背景：由于之前在腾讯购买的云服务器过期，续费的价格虚高，导致最后放弃使用，之前部署的一套Jenkins环境丢失，刚好朋友的阿里云服务器一直处于置灰状态，拿过来重新搭一遍环境



### 前提

需要搭建好JDK和Maven环境，这里就不做赘述。

同时去下载jenkins的war包，这里不建议在docker里面部署jenkins，很多坑。

如果是直接在本地下载的，需要上传到服务器，上传命令rz，这里如果提示命令不存在，则需要安装lszrz。

```shell
#安装lszrz插件
yum install -y lszrz
#上传指令
rz
#下载指令
sz
```

<!-- more -->

### 跑起来

先把下载好/上传好的jenkins.war执行起来。

```shell
#后台运行，并输出日志到out.log
nohup java -jar jenkins.war > out.log &
```

如果没有意外，在浏览器输入对应的ip+端口8080，即可以看到jenkins正在初始化。

首先第一步是要输入管理员的账号密码，默认密码是在目录/data/jenkins_home/secrets/initialAdminPassword

```shell
#如果不知道密码在哪里，可以先查一下，这里选择在根目录下通过文件名称查询
find / -name initialAdminPassword
#查到在哪里后，将里面的内容复制出来
cat /data/jenkins_home/secrets/initialAdminPassword
#拿到密码在页面上输入，即可进入主页面
```

第二步是安装所需要的插件，这里额外提一个插件git parameter，这个可以在构建项目的时候选择构建的分支。



第三步是初始化一些全局配置，如maven、jdk、时间等。

![image-20210412151330194](image-20210412151330194.png)

![image-20210412151402045](image-20210412151422095.png)

![image-20210412151440186](image-20210412151440186.png)

还有就是配置时间，如果不配置的话jenkins上显示的时间不正确，我们一般都是用的上海的时区,在【系统管理】->【脚本命令行】运行下面的命令

```shell
System.setProperty('org.apache.commons.jelly.tags.fmt.timeZone', 'Asia/Shanghai')
```

![image-20210412151759256](image-20210412151759256.png)

### 部署项目

先创建一个流水线项目

![image-20210412145624401](image-20210412145624401.png)

接下来是配置预检分支，配置流水线步骤。

在参数化构建过程里面，我们选择一个git参数，这个参数是作为后面我们部署项目所使用的分支。

![image-20210412145935953](image-20210412145935953.png)

![image-20210412150024211](image-20210412150024211.png)



接下来配置流水线工作，先配置远程仓库

![image-20210412150216717](image-20210412150216717.png)

![image-20210412150246555](image-20210412150246555.png)



脚本路径我这里使用的是项目根目录下的文件，名称为Jenkinsfile，也可以直接用在线脚本的形式。

这里我填一个坑，就是上面截图最后的“轻量级检出”，如果勾选了这个选项，在上面指定的分支会获取不到，这里我检查了很久，因为看了其他博客这里配置是没问题的，后来在stack overflow查到了我想要的答案，就是不要勾选这里。

对应的Jenkinsfile我也贴出来，如下

```shell
        def devUser = 'root'

        def project = 'guns'

        pipeline {

            agent any

            parameters {

                    gitParameter branchFilter: 'origin/(.*)', defaultValue: 'master', name: 'BRANCH', type: 'PT_BRANCH'

             }

            stages {

                stage('代码更新') {

                    steps {
                    #这里的credentialsId在jenkins的全局凭据里面可看到
                        git branch: '$BRANCH',credentialsId: 'huanglifu',
                            url: 'https://gitee.com/rifu520/guns.git'
                            
                    }

                }

                stage('构建代码') {

                    steps {

                        sh 'mvn clean compile'

                    }

                }

                stage('打包服务') {

                    steps {

                        echo '开始打包'
                        
                        sh 'mvn package -Dmaven.test.skip=true'
                        
                        echo '打包完成'

                    }

                }

                stage('创建镜像') {

                    steps {

                        sh """
							cd guns-main

                            mv -f target/*.jar ./${project}.jar

                            docker build -t ${project}:$BUILD_NUMBER .

                            rm -f ./${project}.jar

                        """

                    }

                }

                stage('部署生产环境') {

                    steps {

                        sh """

                            docker  rm -f ${project} | true

                            docker  run -d --name ${project} -p 9090:9090 -v /home/service/guns:/guns -e "spring.profiles.active=dev"  --restart=always ${project}:$BUILD_NUMBER

                        """

                    }

                }

            }

        }
```





以下是dockerfile

```shell
FROM java:8-jre

# 作者
MAINTAINER rifu

# 时区
ENV TZ Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone; mkdir -p guns

# 端口
EXPOSE 9090

# 应用包
ADD guns.jar /usr/local/

# 启动命令
ENTRYPOINT java -Xms512m -Xmx512m \
-XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=128m \
-XX:+PrintGCDetails -XX:+PrintGCDateStamps -Xloggc:guns/gc.log \
-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=guns/heap.log \
-XX:ErrorFile=guns/hs_err_pid%p.log \
-jar /usr/local/guns.jar \

```

