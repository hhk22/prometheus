

## Prometheus Operator 가 Control Plane들의 Metric을 수집하게 하는 방법 

각 Static Pod들의 manifest에서 --bind-address=127.0.0.1 부분을 0.0.0.0 으로 수정한다. 

/etc/kubernetes/manifests/kube-controller-manager.yaml  
/etc/kubernetes/manifests/kube-scheduler.yaml
/etc/kubernetes/manifests/etcd.yaml

기존의 etcd의 2379 port (인증서 까다로움) 을 2381 port로 수정해서 설정하는 방법

```
#upt-kube-etcd.yaml
kubeEtcd:
  service:
    enabled: true  
    port: 2381
    targetPort: 2381
```

해당파일을 helm upgrade시 설정해줌. 

```
helm upgrade prometheus-stack edu/kube-prometheus-stack \
    --set ...
    ...
    ...
    -f ~/upt-kube-etcd.yaml
```

그러고서, 

`--listen-metrics-urls=...` 해당 부분을 다음과 같이 수정한다.  
`--listen-metrics-urls=http://0.0.0.0:2381`


## Service Monitor

1. 사용자가 Service Monitor를 배포
2. Prometheus Operator에 selector에 있는 Service Monitor만을 secret에 Prometheus Server Config에 작성. 
3. Prometheus Server가 Config를 자동으로 (Reloader) 로 업데이트. 

4. Prometheus Server가 Kube-Api-server에게 경로 요청
5. api가 (labels, ns, port) 랑 매칭된 service를 찾고, 거기에 해당하는 endpoints들에 대한 경로를 받아서 Prometheus Server에 반환.
6. Prometheus Server는 해당 경로에 metric정보를 수집. 


### Nginx Service Monitor

```
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  labels:
    app: nginx
    release: prometheus-stack
  name: nginx
  namespace: monitoring
spec:
  namespaceSelector:
    any: true
  selector:
    matchLabels:
      app: nginx
  endpoints:
  -  port: metrics
```

여기서 특별한 것은 `release: prometheus-stack` 부분이다. 해당 Label이 있는 serviceMonitor들만 Prometheus Operator가 metric으로 등록해줄 수 있습니다. (namespace도 동일해야함.)  

또, 특이한것은 endpoints.port 부분이 port 번호가 아닌, port name 을 기준으로 한다는것. 

nginx export를 배포하고서, 

```
apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  type: LoadBalancer
  loadBalancerIP: "192.168.1.84"
  ports:
  - name: web
    port: 80
    targetPort: 8080
  - name: metrics
    port: 9113
    targetPort: 9113
  selector:
    app: nginx
```

해당 부분처럼, name: metrics 부분을 serviceMonitor쪽 부분에 연결해주면, operator가 자동으로 target으로 인식하고 metric을 수집하기 시작한다. 

여기선, <pod_ip>:9113/metrics쪽으로 metric 수집이 가능해야한다. 


## Pod Monitor


작성방식이나, 수집방식은 Service Monitor와 비슷하다. Target이 Service가 아닌 Pod인것일 뿐.. 

```
apiVersion: monitoring.coreos.com/v1
kind: PodMonitor
metadata:
  labels:
    app: metallb
    release: prometheus-stack
  name: metallb
  namespace: monitoring
spec:
  namespaceSelector:
    any: true
  selector:
    matchLabels:
      app: metallb
  podMetricsEndpoints:
    - port: monitoring
```


## Pod Monitor with MYSQL and StatefulSet

```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
  labels:
    app: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  serviceName: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0.30
        ports:
        - name: mysql
          containerPort: 3306
        env:
         - name: MYSQL_ROOT_PASSWORD
           value: root
        volumeMounts:
          - name: data
            mountPath: /var/lib/mysql
      - name: mysqld-exporter
        image: prom/mysqld-exporter:v0.14.0
        ports:
        - name: metrics
          containerPort: 9104
        env:
         - name: DATA_SOURCE_NAME
           value: "root:root@(mysql:3306)/"
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
---
apiVersion: v1
kind: Service
metadata:
  name: mysql
  labels:
    app: mysql
spec:
  ports:
  - name: mysql
    port: 3306
  selector:
    app: mysql
```





