
Gauge : 특정 시점의 값을 표현하는 값  
Counter : 누적된 값을 표현하는 값  
Summary : 특정 구간내에 있는 빈도  
> ex) prometheus_target_interval_length_seconds  

Histogram : 사전에 미리 정의한 구간내에 있는 빈도  

### Label Matcher

```
node_memory_Active_bytes{node="m-k8s"}
node_memory_Active_bytes{node!="m-k8s"}
node_memory_Active_bytes{node=~"w.+"}
node_memory_Active_bytes{node=~"m-k8s|w1-k8s"}
node_memory_Active_bytes{node!~"m-k8s|w1-k8s"}
```

### Binary Operator

```
node_memory_Active_bytes / 1024 / 1024
kube_pod_container_status_restarts_total > 3
kube_pod_container_status_terminated > 0 or kube_pod_container_status_waiting > 0
```

### Aggregation Operator

```
topk(3, node_cpu_seconds_total)
bottomk(3, node_cpu_seconds_total)
bottomk(3, node_cpu_seconds_total > 0)

avg(node_cpu_seconds_total)
avg(node_cpu_seconds_total) by (node)
avg(node_cpu_seconds_total{mode="user"} by (node))
sum(kubelet_http_requests_total) withtout (node)
>> node를 제외한 값들별로 통계. 
>> path="metrics", node="m-k8s", ...  --> 이것들끼리 sum 결과
>> path="metrics/cadvisor", node="m-k8s", ... --> 이것들끼리 sum 결과

```

### Range Vector

```
node_memory_Active_bytes[1m]
>> 현재 시점에서 1분동안의 metircs
```

### Modifider

```
node_memory_Active_bytes offset 1m
node_memory_Active_bytes offset @<unix_time>
```

### Custom Functions

```
rate(node_cpu_seconds_total{node="m-k8s", mode="system"}[5m])
irate(node_cpu_seconds_total{node="m-k8s", mode="system"}[5m])
predict_linear(node_memory_Active_bytes[5m], 60*60*2) / 1024 / 1024
```













