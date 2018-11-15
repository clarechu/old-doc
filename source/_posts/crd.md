---
title: crd
catalog: true
date: 2018-11-12 16:17:28
header-img: "7vLMRNQ.jpg"
tags:
- crd
- k8s
subtitle: crd资源的整合
---

# 首先创建资源

```bash
#如果在openshift中创建资源
oc apply -f deploymentconfig-crd.yaml

#若在k8s中创建资源
kubectl apply -f deploymentconfig-crd.yaml
```

crd 资源demo

```yaml
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: builds.mio.io
spec:
  group: mio.io
  version: v1alpha1
  names:
    kind: Build
    plural: builds
  scope: Namespaced
```

# pipelineconfig 资源

```yaml
apiVersion: mio.io/v1alpha1
kind: PipelineConfig
metadata:
  creationTimestamp: 2018-10-15T08:12:49Z
  generation: 0
  name: java
  namespace: templates
  resourceVersion: "125861171"
  selfLink: /apis/mio.io/v1alpha1/namespaces/templates/pipelineconfigs/java
  uid: 1b3cbbc0-d052-11e8-b78c-005056935c80
spec:
  app: ""
  dockerRegistry: ""
  events:
  - eventTypes: buildPipeline
    name: java
  - eventTypes: deploy
    name: java
  - eventTypes: deploy
    name: java-test
  - eventTypes: service
    name: java
  - eventTypes: gateway
    name: java
  profile: dev
  version: v2
status: {}
```

# buildconfig 资源

```yaml
apiVersion: mio.io/v1alpha1
kind: BuildConfig
metadata:
  creationTimestamp: 2018-10-23T08:29:45Z
  generation: 0
  name: java
  namespace: templates
spec:
  baseImage: docker.vpclub.cn/hidevopsio/hinode-java-jar:1.22 # hinode镜像地址
  cloneConfig:
    branch: master #拉取代码的分支
    dstDir: /opt/app-root/src/vpclub # 默认地址
    url: https://gitlab.vpclub.cn # 拉取 代码的基础地址
  codeType: java # 代码类型
  compileCmd:
  - commandName: pwd
  - Script: |-
      mvn clean package -U -Dmaven.test.skip=true -Djava.net.preferIPv4Stack=true
      if [[ $? == 0 ]]; then
        echo 'Build Successful.'
      else
        echo 'Build Failed!'
        exit 1
      fi
    execType: script
  - commandName: pwd
  - Script: ls
    execType: script
  deployData:
    envs: # deploy
      CODE_TYPE: java
      DOCKER_API_VERSION: "1.24" # docker api 版本号
      MAVEN_MIRROR_URL: http://nexus.vpclub.cn/repository/maven-public/ #maven仓库地址
    hostPathVolume:
      /var/lib/docker: /var/lib/docker #挂载目录
      /var/run/docker.sock: /var/run/docker.sock
    ports:
    - 8080 #暴露端口
    - 7575
    replicas: 1 # replicas pod的数量
  dockerAuthConfig:
    password: #docker鉴权所需要的账号密码
    username: unused
  dockerFile: # 制作 镜像的 dockerfile
  - FROM clarechu/base-image-java:0.1
  - ENV  TZ="Asia/Shanghai"
  - ENV  APP_OPTIONS="-Xms128m -Xmx512m -Xss512k"
  - ENV   APP_OPTIONS="-Xms128m -Xmx512m -Xss512k"
  - USER 0
  - RUN  useradd -u 1002 -r -g 0 -d ${HOME} -s /sbin/nologin -c "Default Application
    User" java
  - COPY ./app.jar ${HOME}
  - RUN chown -R 1001:0 ${HOME}
  - USER 1002
  - EXPOSE 8080
  - EXPOSE 7575
  - ENTRYPOINT ["sh","-c","java -jar $HOME/app.jar $APP_OPTIONS"]
  dockerRegistry: docker-registry-default.app.vpclub.io #上传的镜像仓库地址
  tasks: #流程
  - name: createService #创建 hinode 的service
  - name: deployNode # 创建deployment
  - name: clone # clone代码
  - name: compile # 编译clone下来的代码
  - name: buildImage # 编译镜像
  - name: pushImage # 推镜像
  - name: deleteDeployment # 删除hinode资源 和 service
status:
  lastVersion: 1
```

# deploymentconfig 资源

```yaml
apiVersion: mio.io/v1alpha1
kind: DeploymentConfig
metadata:
  name: java-test
  namespace: templates
spec:
  dockerRegistry: docker-registry.default.svc:5000
  env:
  - name: starter
    value: jav -jar
  - name: TZ
    value: Asia/Shanghai
  - name: APP_OPTIONS
    value: -Xms128m -Xmx512m -Xss512k
  - name: SPRING_PROFILES_ACTIVE
    value: test
  envType:
  - remoteDeploy
  - deploy
  fromRegistry: docker-registry-default.app.vpclub.io
  port:
  - containerPort: 8080
    name: tcp-8080
    protocol: TCP
  profile: test #部署环境
status:
  lastVersion: 1
```

# sourceconfig 资源

```yaml
apiVersion: mio.io/v1alpha1
kind: SourceConfig
metadata:
  name: java-source
spec:
  scm:
    apiVersion: v3
    ref: master
    token: ""
    url: http://gitlab.vpclub:8022
  sourceCode:
  - content: <packaging>jar</packaging>
    path: pom.xml
    type: java
status:
  metadata: {}
```

# gatewayconfig 资源

```yaml
apiVersion: mio.io/v1alpha1
kind: GatewayConfig
metadata:
  clusterName: ""
  creationTimestamp: 2018-10-26T09:01:51Z
  generation: 0
  name: java
  namespace: templates
spec:
  eventType: null
  hosts:
  - dev.vpclub.cn
  httpIfTerminated: false
  httpsOnly: false
  kongAdminUrl: http://kong-admin-kong-dev.app.vpclub.io
  preserveHost: true
  profile: dev
  retries: "5"
  stripUri: true
  upstreamConnectTimeout: 60000
  upstreamReadTimeout: 60000
  upstreamSendTimeout: 60000
  upstreamUrl: ""
  uris: null
status:
  metadata: {}
```

# serviceconfig 资源

```yaml
apiVersion: mio.io/v1alpha1
kind: ServiceConfig
metadata:
  clusterName: ""
  creationTimestamp: 2018-10-26T07:31:26Z
  generation: 0
  labels:
    profile: dev
  name: java
  namespace: templates
spec:
  ports:
  - name: http-8080
    port: 8080
    protocol: TCP
    targetPort: 8080
  - name: http-7575
    port: 7575
    protocol: TCP
    targetPort: 7575
status:
  metadata: {}
```
