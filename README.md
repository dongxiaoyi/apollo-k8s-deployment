# k8s部署apollo 1.6.0
#k8s #apollo
*# 使用方法*

参考：https://github.com/ctripcorp/apollo/tree/master/scripts/apollo-on-kubernetes

*## 部署规划*

1.本次部署采用apollo 1.6.0版本

2.部署环境： apollo 开启了4 个环境, 即 dev、test-alpha、test-beta、prod

3.各环境服务pod数量规划：
(1)dev
config-server:1
admin-server:1
(2)prod
config-server:1
admin-server:1
(3)test-alpha
config-server:1
admin-server:1
(4)test-beta
config-server:1
admin-server:1
(5)其他
portal-server:1

*## 一、构建镜像*

*### 1.1 获取 apollo 压缩包*
从 https://github.com/ctripcorp/apollo/releases 下载预先打好的 java 包 <br/>
例如你下载的是: <br/>
apollo-portal-1.6.0-github.zip <br/>
apollo-adminservice-1.6.0-github.zip <br/>
apollo-configservice-1.6.0-github.zip <br/>

*### 1.2 解压压缩包, 获取程序 jar 包*
**>**以下内容已完成，如需其他版本的apollo，参考替换
**-**解压 apollo-portal-1.6.0-github.zip <br/>
获取 apollo-portal-1.6.0.jar, 重命名为 apollo-portal.jar, 将整个项目内容放到 apollo-on-kubernetes/apollo-portal-server
**-**解压 apollo-adminservice-1.6.0-github.zip <br/>
获取 apollo-adminservice-1.6.0.jar, 重命名为 apollo-adminservice.jar, 将整个项目内容放到 apollo-on-kubernetes/apollo-admin-server
**-**解压 apollo-configservice-1.6.0-github.zip <br/>
获取 apollo-configservice-1.6.0.jar, 重命名为 apollo-configservice.jar, 将整个项目内容放到 apollo-on-kubernetes/apollo-config-server

*### 1.3 build image*
需要分别为alpine-bash-3.8-image，apollo-config-server，apollo-admin-server和apollo-portal-server构建镜像。

以 build apollo-config-server image 为例, 其他类似

```bash
apollo-config-server$ tree -L 2
.
├── apollo-configservice.conf
├── apollo-configservice.jar
├── config
│   ├── application-github.properties
│   └── app.properties
├── Dockerfile
├── entrypoint.sh
└── scripts
    └── startup-kubernetes.sh
```
注意：
（1）镜像制作有权限问题，请参考： [https://docs.openshift.com/container-platform/4.1/openshift_images/create-images.html](https://docs.openshift.com/container-platform/4.1/openshift_images/create-images.html) 
需要给操作目录添加权限：
```dockerfile
RUN chgrp -R 0 /some/directory && \
    chmod -R g=u /some/directory
```

build image：
```bash
# 在 apollo-on-kubernetes/apollo-config-server 路径下
$ docker build -t harbor.gzky.com/devops-example/apollo-config-server:v1.6.0 .
```
最终打包的镜像如下：
```
harbor.gzky.com/devops-example/apollo-portal-server:v1.6.0
harbor.gzky.com/devops-example/apollo-admin-server:v1.6.0
harbor.gzky.com/devops-example/apollo-config-server:v1.6.0
harbor.gzky.com/devops-example/apollo-alpine-bash:3.8
```
push image <br/>
将 image push 到你的 docker registry, 例如harbor
```bash
$ docker push harbor.gzky.com/devops-example/apollo-portal-server:v1.6.0
$ docker push harbor.gzky.com/devops-example/apollo-admin-server:v1.6.0
$ docker push harbor.gzky.com/devops-example/apollo-config-server:v1.6.0
$ docker push harbor.gzky.com/devops-example/apollo-alpine-bash:3.8
```

*## 二、Deploy apollo on kubernetes*

*### 2.1 部署 MySQL 服务*
**>**使用外部MySQL， 部署步骤略。本次使用公司mysql 5.7，服务器地址：192.168.6.170 

*### 2.2 导入 MySQL DB 文件*
本次 config-server、admin-server、portal-server 使用上述同一个 MySQL 服务的不同数据库

本次的 apollo 只开启了 4 个环境, 即 dev、test-alpha、test-beta、prod, 在你的 MySQL 中导入 apollo-on-kubernetes/db 下的文件即可

数据库导入参考：
```sql
source db/config-db-dev/apolloconfigdb.sql
source db/config-db-prod/apolloconfigdb.sql
source db/config-db-test-alpha/apolloconfigdb.sql
source db/config-db-test-beta/apolloconfigdb.sql
source db/portal-db/apolloportaldb.sql
```

需要提前创建用户，参考：
```sql
create user 'apollo'@'%' identified by 'apollo';
GRANT all  privileges ON ApolloPortalDB.* TO 'apollo'@'%';
GRANT all  privileges ON DevApolloConfigDB.* TO 'apollo'@'%';
GRANT all  privileges ON ProdApolloConfigDB.* TO 'apollo'@'%';
GRANT all  privileges ON TestAlphaApolloConfigDB.* TO 'apollo'@'%';
GRANT all  privileges ON TestBetaApolloConfigDB.* TO 'apollo'@'%';
flush privileges;
```

如果有需要, 你可以更改 eureka.service.url 的地址, 格式为 http://config-server-pod-name-index.meta-server-service-name:8080/eureka/ , 例如 http://statefulset-apollo-config-server-dev-0.service-apollo-meta-server-dev:8080/eureka/

*### 2.2 Deploy apollo on kubernetes*
根据 `kubernetes/` 目录部署。
部署顺序：

（1）apollo-env-dev/service-mysql-for-apollo-dev-env.yaml

（2）apollo-env-dev/service-apollo-config-server-dev.yaml

（3）apollo-env-dev/service-apollo-admin-server-dev.yaml

（4）apollo-env-prod/service-mysql-for-apollo-prod-env.yaml

（5）apollo-env-prod/service-apollo-config-server-prod.yaml

（6）apollo-env-prod/service-apollo-admin-server-prod.yaml

（7）apollo-env-test-alpha/service-mysql-for-apollo-test-alpha-env.yaml

（8）apollo-env-test-alpha/service-apollo-config-server-test-alpha.yaml

（9）apollo-env-test-alpha/service-apollo-admin-server-test-alpha.yaml

（10）apollo-env-test-beta/service-mysql-for-apollo-test-beta-env.yaml

（11）apollo-env-test-beta/service-apollo-config-server-test-beta.yaml

（12）apollo-env-test-beta/service-apollo-admin-server-test-beta.yaml

（13）service-apollo-portal-server.yaml

注意：

(1) 部署时注意yaml部署配置中的注释，修改正确的namespace地址。

(2) 部署时注意修改为正确的image地址。

(3) 部署时注意修改为正确的数据库地址。

*### 2.4 访问 apollo service*

**-**server 端(即 portal) <br/>
&nbsp;&nbsp;&nbsp;&nbsp;kubernetes-master-ip:30001

**-**client 端, 在 client 端无需再实现负载均衡 <br/>
Dev<br/>
&nbsp;&nbsp;&nbsp;&nbsp;kubernetes-master-ip:30002 <br/>
Test-Alpha <br/>
&nbsp;&nbsp;&nbsp;&nbsp;kubernetes-master-ip:30003 <br/>
Test-Beta <br/>
&nbsp;&nbsp;&nbsp;&nbsp;kubernetes-master-ip:30004 <br/>
Prod <br/>
&nbsp;&nbsp;&nbsp;&nbsp;kubernetes-master-ip:30005 <br/>

