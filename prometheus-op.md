
## Prometheus BlackBox

BlackBox의 메트릭은 /Probe 를 통해서 수집됨.  

아쉽게도, ServiceMonitor,PodMonitor 들은 Metric Path를 설정할 수 없음. --> Probe crd를 사용해야함. 

설정 프로세스

1. 사용자가 Probe crd를 작성. 
2. Prometheus Operator를 Matched Labels에 대한 probe crd를 secret에 작성하고, 그것을 Prometheus server config에 작성.
3. Prometheus Server는 해당 label에 대한 endpoints 들을 Prometheus Blackbox에 요청. 
4. Prometheus Blackbox 는 앤드포인트를 모니터링하고 그에 대한 결과를 확인하고 응답해줌. 


## Prometheus Operator Rules

1. 사용자가 PrometheusRule(CRD)을 작성
2. Prometheus Rule은 기록규칙,경보규칙을 생성함. 
3. Prometheus Operator가 일치하는 namespace,labels 들의 Rule을 configMap에 직접작성. 
4. Prometheus Server가 ConfigMap을 읽고 실행됨. 

```
apiVersion: monitoring.coreos.com/v1 
kind: PrometheusRule
metadata:
  name: kubernetes-containers-metric
  namespace: monitoring
  labels:
    release: prometheus-stack
spec:
  groups:
    - name: kubernetes-containers-metric
      interval: 10s
      rules:
      - record: container:memory_working_set:topk3
        expr: topk(3,sum(container_memory_working_set_bytes{pod!=""}/1024/1024) by (pod))
```

> kubectl apply -f \<patch-file\>

이렇게 되면, operator가 해당 정보를 읽고서, configMap에 해당 정보를 update함. 

그렇게 되면, prometheus server가 reloader를 통해서 자동으로 update함. 


## Prometheus Operator AlertRules

alertManager의 정보는 secret으로 관리되고 있음. 
PrometheusRule은 cm에서 관리되고 있음. 

```
apiVersion: monitoring.coreos.com/v1 
kind: PrometheusRule
metadata:
  name: prometheus-operator-down
  namespace: monitoring
  labels:
    release: prometheus-stack
spec:
  groups:
  - name: prometheus-operator
    rules:
    - alert: PrometheusOperatorDown
      annotations:
        title: "Prometheus operator DOWN!!!"
        description: "prometheus-operator is out of service status"
        summary: "[P1, Critical]: Prometheus operator has been shutdown or evicted status unexpectedly."
      expr: |
        ((sum(prometheus_operator_ready)) or vector(0)) == 0
      for: 30s
```


```
alertmanager:
  enabled: true
  service:
    type: "LoadBalancer"
    loadBalancerIP: "192.168.1.95"
  config:
    global:
      resolve_timeout: 10m
      slack_api_url: Slack-URL
    inhibit_rules:
    receivers:
    - name: default-receiver
    - name: slack
      slack_configs:
      - send_resolved: true
        title: '[{{.Status | toUpper}}] {{ .CommonLabels.alertname }}'
        text: |
          *Description:*
          {{ .CommonAnnotations.description }}
    route:
      group_by:
      group_interval: 1m
      group_wait: 10s
      receiver: slack
      repeat_interval: 5m
      routes:
      - receiver: slack
```




