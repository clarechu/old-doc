---
layout: deployment
title: hiadmin
catalog: true
date: 2018-11-12 10:55:46
tags: 
- k8s
- deployment
- devopsio
subtitle: "hiadmin 部署手册"
header-img: "7vLMRNQ.jpg"
---

# 部署 hiadmin

hiadmin该服务主要负责启动pipeline，任务调度， 针对k8s 资源`crd`进行控制， 和部署流程的管控。

1. 创建hiadmin的 deployment
    因为需要在容器里面使用docker，则该项目需要挂载 /var/lib/docker ， /var/run/docker.sock
    镜像上传到 docker.vpclub.cn
    ```yaml
    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      annotations:
      labels:
        app: hiadmin
        cluster: hidevopsio
      name: hiadmin
      namespace: hidevopsio
    spec:
      replicas: 1
      selector:
        matchLabels:
        app: hiadmin
        cluster: hidevopsio
      strategy:
        rollingUpdate:
        maxSurge: 1
        maxUnavailable: 1
        type: RollingUpdate
      template:
        metadata:
        creationTimestamp: null
        labels:
          app: hiadmin
          cluster: hidevopsio
        spec:
        nodeName: paasm1
        containers:
        - env:
          - name: TZ
            value: Asia/Shanghai
          - name: APP_PROFILES_ACTIVE
            value: dev
          - name: SCM_URL
            value: https://gitlab.vpclub.cn
          image: docker.vpclub.cn/hidevopsio/hiadmin:v5
          imagePullPolicy: Always
          name: hiadmin
          ports:
          - containerPort: 7575
            protocol: TCP
          - containerPort: 8080
            protocol: TCP
          resources: {}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          volumeMounts:
          - mountPath: /var/lib/docker
            name: volume1
          - mountPath: /var/run/docker.sock
            name: volume2
        dnsPolicy: ClusterFirst
        restartPolicy: Always
        schedulerName: default-scheduler
        securityContext: {}
        terminationGracePeriodSeconds: 30
        volumes:
        - hostPath:
          path: /var/lib/docker
          type: ""
        name: volume1
        - hostPath:
          path: /var/run/docker.sock
          type: ""
        name: volume2
    ```

2. 创建hiadmin的service
    hiadmin 开启需要开启8080 和 7575 端口
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: hiadmin
      namespace: hidevopsio
    spec:
      ports:
      - name: 8080-tcp
        port: 8080
        protocol: TCP
        targetPort: 8080
     - name: 7575-tcp
        port: 7575
        protocol: TCP
        targetPort: 7575
      selector:
        app: hiadmin
      sessionAffinity: None
      type: ClusterIP
    status:
      loadBalancer: {}
    ```

3. 配置nginx通过网关可以访问服务
    hiadmin 需要websocket所以nginx网关需要添加 `proxy_http_version 1.1; proxy_set_header Upgrade $http_upgrade; proxy_set_header Connection "upgrade"; proxy_set_header Origin ""`.

   ```yaml
   server {
    listen      80;
    server_name nginx-gateway.app.vpclub.io;

    access_log  /dev/stdout  main;
    error_log   /dev/stderr;
    client_max_body_size 200m;
    location = /favicon.ico {
        return 204;
        access_log     off;
        log_not_found  off;
    }

    # limit_req zone=REQ burst=2;
    # limit_conn CONN 10;
    location /demo/hiadmin/ {
        client_max_body_size 200m;
        proxy_pass http://hiadmin-demo.app.vpclub.io/; 
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    location /demo/websocket/ {
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Origin "";
        client_max_body_size 200m;
        proxy_pass http://hiadmin-demo.app.vpclub.io/;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        }

    }
   ```

# 部署hinode

hinode的作用主要是在于针对代码的编译，比如java的 `mvn clean install`, nodejs的`npm install && npm run build:dev`等等操作。另外还有制作镜像，推镜像等过程。

image： `docker.vpclub.cn/hidevopsio/hinode-java-jar:1.22`

hinode 是由`hiadmin`发送指令进行操作的
