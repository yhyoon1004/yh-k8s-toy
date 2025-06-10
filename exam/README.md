## 1.다음 조건에 맞는 Deployment를 생성하세요.
1) 도커 허브의 nginx 이미지를 사용합니다.
2) Deployment의 이름은 nginx-web으로 설정합니다.
3) Pod의 복제본은 3개로 설정합니다.
4) 명령어를 이용하여 yaml 파일을 생성하고, yaml 파일을 이용하여 배포합니다.
5) 생성된 Deployment와 Pod가 제대로 작동하는지 확인하는 명령어를 실행해 결과를 확인하세요.

```shell
  kubectl create deployment nginx-web --image nginx:latest --replicas 3 --dry-run=client -o yaml > nginx-web.yaml
```
```shell
  kubectl apply -f nginx-web.yaml
```
---

## 2. 도커 허브의 httpd:latest 이미지를 사용합니다.
1) Deployment 이름은 apache-web으로 설정합니다.
2) Pod의 복제본은 2개로 설정합니다.
3) LoadBalancer 타입의 Service를 생성합니다.
4) Service 이름은 apache-service입니다.
5) 80번 포트를 통해 웹페이지에 접근할 수 있어야 합니다.
6) Deployment와 Service를 생성한 뒤, localhost로 웹 브라우저에서 접근해봅니다.
```shell
   kubectl create deployment apache-web --image httpd:latest --replicas 2 --dry-run=client -o yaml > apache-web.yaml
```
```shell
   kubectl apply -f apache-web.yaml
```
```shell
  kubectl create service loadbalancer apache-service --tcp=80:80  -o yaml > apache-service.yaml
```
```shell
   vi apache-service.yaml
   ~~추가/수정~~ 
     selector:
       app: apache-web
   ~~~
```
```shell
   kubectl apply -f apache-service.yaml
```
---

## 3. `2번` 문제에서 생성된 Deployment를 롤링 업데이트를 사용하여 새로운 이미지를 적용하고, 중단 없이 업데이트를 완료하세요.
1) 기존에 생성한 apache-web Deployment를 사용합니다.
2) Deployment의 이미지를 httpd:2.4-alpine으로 변경합니다.
3) 롤링 업데이트를 수행하여 새로운 이미지로 변경된 Deployment를 생성합니다.
4) 업데이트 과정에서 다운타임이 없어야 합니다.
5) 업데이트 완료 후, 새로운 이미지가 적용되었는지 확인하세요.

```shell
   vi apache-web.yaml
   ~~추가/수정~~
          metadata:
         creationTimestamp: null
         labels:
           app: apache-web
       spec:
         containers:
         - image: httpd:2.4-alpine #여기 수정 latest > httpd:2.4-alpine
           name: httpd
           resources: {}
   status: {}

   ~~~~
```
```shell
   kubectl apply -f apache-web.yaml
```
```shell
   kubectl exec -it deployments/apache-web -c httpd -- sh
```
```shell
   httpd -v
   uname -a
```

`short cut` : 
```shell
   kubectl create deployment apache-web --image=httpd:latest --replicas=2 &&\
   kubectl expose deployment apache-web --name=apache-service --type=LoadBalancer --port=80
```

---

## 4. ConfigMap을 생성하여 Deployment에서 환경 변수를 주입받아 사용하도록 구성하세요.
1) ConfigMap 이름은 app-config로 설정합니다.
2) ConfigMap에는 다음 키-값 쌍이 포함되어야 합니다:
   ``` 
   APP_ENV=production
   APP_VERSION=1.0
   ```
3) Deployment 이름은 configmap-demo로 설정합니다.
4) 도커 허브의 nginx 이미지를 사용하여 Pod를 배포합니다.
5) Pod에서 ConfigMap 값을 환경 변수로 주입받아 활용하도록 설정합니다.
6) 생성된 Pod 내부에서 환경 변수가 설정되었는지 확인하세요.

```shell
   kubectl create configmap app-config --from-literal=APP_ENV=production --from-literal=APP_VERSION=1.0 --dry-run=client -o yaml > app-config.yaml
```
```shell
   vi configmap-demo.yaml
   ~~추가/수정~~
    spec:
      containers:
      - image: nginx:latest
        name: nginx
        resources: {}
        envFrom:
        - configMapRef:
            name: app-config
   ~~~~
```
Deployment 재시작
```shell
  kubectl rollout restart deployment configmap-demo
```

```shell
kubectl exec -it deployments/configmap-demo -- bash
#env
```