
### cAdvisor

prefix = container_

kubelet에 내장되어있는 cAdvisor를 통해서 container에 대한 metric을 수집. 

테스트

metric : `container_network_receive_bytes_total{pod="chk-hn"}`

test yaml

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: chk-hn
  name: chk-hn
spec:
  containers:
  - image: sysnet4admin/chk-hn
    name: chk-hn
```

해당 container로 curl를 계속 날리게되면, 해당되는 metirc의 counter가 계속 증가하는것을 관찰할 수 있음. 


### nodeExporter

- 각 노드에 prometheus

- 다음과 같은 폴더에 있는 정보를 보내줌. 
1. /sys
2. /proc

- 부하주기

> stress --vm 5 --vm-bytes 10M --timeout 150s

metirc: `node_memory_Active_bytes{node="w2-k8s"}`

![img](https://github.com/hhk22/prometheus/blob/master/images/node_exporter_memory_surge.png)

> stress --cpu 2 --timeout 300s

metric: `node_cpu_seconds_total{node="w2-k8s", mode="user"}`

![img](https://github.com/hhk22/prometheus/blob/master/images/node_exporter_cpu_surge.png)


### kube-state-metric(object)

많은 object들.. (deploy, configmap, secret, ..) 에 대한 정보가 kube-api-server에 저장되어 있다.  

kube-state-metrics라는 pod가 해당 kube-api-server에 /metrics 를 얻어서 prometheus가 이를 이용하는 구조.  

`prefix = kube_`

### application-metric

application 단에서 metric을 expose해서 target으로 prometheus에서 지정되면 그것을 수집할 수 있음. 

nginx을 예로 들면, 

1. Nginx Container (8080 port)
2. Nginx Prometheus Container (9113 port)

이 두개를 배포하게 되면, 
Prometheus Conatiner가 8080 port를 통해 Application 정보를 얻어오고, Prometheus Server가 해당 9113포트를 통해 metirc들을 수집함. 

##### 어떻게 Target으로 지정할 수 있는거지?

```
annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "9113"
```

이런식으로 container pod든, service든, 이렇게 지정되어 있으면, 해당 ip의 port로 scrape을 진행한다.





