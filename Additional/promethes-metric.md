
## Label 재지정 (Native Prometheus)

입력 : source_labels

선택후처리 : regex -> action ( replace, keep, drop, labeldrop, labelkeep, labelmap )

출력 : target_label


```
relabel_configs:
- action: drop
  regex: true
  source_labels:
    - __meta_kubernetes_service_annotation_prometheus_io_exclude
>>>>>
annotations:
    prometheus.io/exclude: "true"
>>>>> 해당 metric은 drop 됨. 
```

```
relabel_configs:
- action: replace
  regex: (.*)
  replacement: $1
  source_labels:
  - __meta_kubernetes_endpoint_port_name
  target_label: portname
>>>>>
prometheus 결과값에 label이 보임.
node_cpu_seconds_total{....., portname="metrics", ....}
>>>>>
```


## Label 재지정 (operator 환경)

```
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  labels:
    app: nginx
    release: prometheus-stack
  name: nginx-relabeling
  namespace: monitoring
spec:
  namespaceSelector:
    any: true
  endpoints:
  - port: metrics
    relabelings:
    - action: keep
      regex: "true"
      replacement: $1
      separator: ;
      sourceLabels:
      - __meta_kubernetes_service_annotation_prometheus_io_scrape
    - action: labelmap
      regex: __meta_kubernetes_service_label_(.+)
      replacement: $1
      separator: ;
    scheme: http
  selector:
    matchLabels:
      app: nginx
```

