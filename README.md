* Argo CD 설치

```bash
$ kubectl create namespace argocd
$ kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/ha/install.yaml
```

운영 서버에서는 HA 버전의 argo CD를 설치하라고 한다.

* 파드조회

```bash
$ kubectl get pod -n argocd

NAME                                                READY   STATUS              RESTARTS   AGE
argocd-application-controller-0                     0/1     ContainerCreating   0          37s
argocd-applicationset-controller-79f97597cb-n7rwm   0/1     ContainerCreating   0          38s
argocd-dex-server-fd9588cbc-6qzh6                   0/1     Init:0/1            0          38s
argocd-notifications-controller-855df7bb69-ljwj2    0/1     ContainerCreating   0          37s
argocd-redis-ha-haproxy-5b75bb98dc-c8dl6            0/1     Pending             0          37s
argocd-redis-ha-haproxy-5b75bb98dc-hr96q            0/1     Init:0/1            0          37s
argocd-redis-ha-haproxy-5b75bb98dc-v57vv            0/1     Pending             0          37s
argocd-redis-ha-server-0                            0/2     Init:0/1            0          37s
argocd-repo-server-7f77b9f7c9-2lgpn                 0/1     Init:0/1            0          37s
argocd-repo-server-7f77b9f7c9-7zfqr                 0/1     Pending             0          37s
argocd-server-bc4b767c6-68758                       0/1     Pending             0          37s
argocd-server-bc4b767c6-h8rgt                       0/1     ContainerCreating   0          37s
```

* 서비스 목록 조회

```bash
$ kubectl get service -n argocd

NAME                                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
argocd-applicationset-controller          ClusterIP   10.102.188.185   <none>        7000/TCP                     78s
argocd-dex-server                         ClusterIP   10.106.78.243    <none>        5556/TCP,5557/TCP,5558/TCP   78s
argocd-metrics                            ClusterIP   10.107.182.193   <none>        8082/TCP                     78s
argocd-notifications-controller-metrics   ClusterIP   10.98.77.46      <none>        9001/TCP                     78s
argocd-redis-ha                           ClusterIP   None             <none>        6379/TCP,26379/TCP           78s
argocd-redis-ha-announce-0                ClusterIP   10.107.160.253   <none>        6379/TCP,26379/TCP           78s
argocd-redis-ha-announce-1                ClusterIP   10.101.127.56    <none>        6379/TCP,26379/TCP           78s
argocd-redis-ha-announce-2                ClusterIP   10.97.148.249    <none>        6379/TCP,26379/TCP           78s
argocd-redis-ha-haproxy                   ClusterIP   10.103.29.72     <none>        6379/TCP                     78s
argocd-repo-server                        ClusterIP   10.108.223.93    <none>        8081/TCP,8084/TCP            78s
argocd-server                             ClusterIP   10.96.104.119    <none>        80/TCP,443/TCP               78s
argocd-server-metrics                     ClusterIP   10.105.217.176   <none>        8083/TCP                     78s
```

* 서비스 상세 조회

```bash
$ kubectl edit svc argocd-server -n argocd

apiVersion: v1
kind: Service
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"labels":{"app.kubernetes.io/component":"server","app.kubernetes.io/name":"argocd-server","app.kubernetes.io/part-of":"argocd"},"name":"argocd-server","namespace":"argocd"},"spec":{"ports":[{"name":"http","port":80,"protocol":"TCP","targetPort":8080},{"name":"https","port":443,"protocol":"TCP","targetPort":8080}],"selector":{"app.kubernetes.io/name":"argocd-server"}}}
  creationTimestamp: "2022-06-02T13:14:01Z"
  labels:
    app.kubernetes.io/component: server
    app.kubernetes.io/name: argocd-server
    app.kubernetes.io/part-of: argocd
  name: argocd-server
  namespace: argocd
  resourceVersion: "617"
  uid: 4ab80875-9400-4299-bd8a-34119a3cf2c1
spec:
  clusterIP: 10.96.104.119
  clusterIPs:
  - 10.96.104.119
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 8080
  - name: https
    port: 443
    protocol: TCP
    targetPort: 8080
  selector:
    app.kubernetes.io/name: argocd-server
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}
```

ClusterIP 타입을 NodePort 타입으로 변경

```bash
$ kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'

$ kubectl edit svc argocd-server -n argocd
apiVersion: v1
kind: Service
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"labels":{"app.kubernetes.io/component":"server","app.kubernetes.io/name":"argocd-server","app.kubernetes.io/part-of":"argocd"},"name":"argocd-server","namespace":"argocd"},"spec":{"ports":[{"name":"http","port":80,"protocol":"TCP","targetPort":8080},{"name":"https","port":443,"protocol":"TCP","targetPort":8080}],"selector":{"app.kubernetes.io/name":"argocd-server"}}}
  creationTimestamp: "2022-06-02T13:14:01Z"
  labels:
    app.kubernetes.io/component: server
    app.kubernetes.io/name: argocd-server
    app.kubernetes.io/part-of: argocd
  name: argocd-server
  namespace: argocd
  resourceVersion: "1184"
  uid: 4ab80875-9400-4299-bd8a-34119a3cf2c1
spec:
  clusterIP: 10.96.104.119
  clusterIPs:
  - 10.96.104.119
  externalTrafficPolicy: Cluster
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - name: http
    nodePort: 31879
    port: 80
    protocol: TCP
    targetPort: 8080
  - name: https
    nodePort: 30866
    port: 443
    protocol: TCP
    targetPort: 8080
  selector:
    app.kubernetes.io/name: argocd-server
  sessionAffinity: None
  type: NodePort
status:
  loadBalancer: {}
```

edit 명령어를 통해 type이 NodePort로 변경된 것을 확인할 수 있다.

* 포트포워딩

```bash
$ kubectl port-forward svc/argocd-server -n argocd 30100:80
```

TODO: port-forward와 네트워크 동작원리 좀 더 이해하기

* argocd 서버 접속

포트포워딩한 주소로 접속한다. https://localhost:30100

이때 계정은 admin이며 비밀번호는 다음 명령어로 base64로 인코딩 된 값(secret에 저장된 값)을 알 수 있다.

```bash
$ kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" ;echo
```

하지만 우리는 디코딩 된 값이 필요하기 때문에 다음 명령어를 통해 디코딩 된 값을 조회한다.

```bash
$ kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```