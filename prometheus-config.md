

### Kube Proxy를 위한 독립적인 Target 설정해보기. 

이런식의 scrape_config정보를 추가해주면 job이 추가된다. 

```
-   job_name: kube-proxy
    honor_labels: true
    kubernetes_sd_configs:
    - role: pod  // search 대상이 pod
    relabel_configs:
    - action: keep
    source_labels:
    - __meta_kubernetes_namespace
    - __meta_kubernetes_pod_name
    separator: '/'  // namespace/pod_name 으로 pod들을 조합. 
    regex: 'kube-system/kube-proxy.+'  // 위의 sep으로 조합된 pod들 대상으로 matching된 애들. 
    - source_labels:
    - __address__  // 해당 pod의 address를 가지고서
    action: replace // replace
    target_label: __address__  // return value 또한 address
    regex: (.+?)(\\:\\d+)?  // ip:port 들을 group으로 묶고,
    replacement: $1:10249 // ip:10249 로 replace
```

### PromQL 기록 규칙. 

명명규칙 `level:metric:operations`

```
data:
    recording_rules.yaml: |
        groups:
            - name: prometheus-recording.rules
              interval: 10s
              rules:
                - record: container:memory_working_set:topk3
                  expr: topk(3, sum(container_memory_working_set_bytes{pod!="}/1024/1024) by (pod))
```

> kubectl patch cm -n monitoring prometheus-server --patch-file \<patch-file\>


### Alert

1. Prometheus를 설치할때, alertManager가 설치되어있어야함. 
2. Slack 에서 수신WebHook가 설치되어있어야함. 

```
data:
  alerting_rules.yml: |
    groups:
    - name: nginx-status.alert
      rules:
      - alert: '[P2] NginxDown'
        for: 30s
        annotations:
          title: 'Nginx pod down unexpectedly'
          description: 'Nginx가 비정상 종료됨. 빠른 조치 필요!'
          summary: '[P2, warnning]: Nginx pod shutdown unexpectedly.'
        expr: |
          (sum(nginx_up) OR vector(0)) == 0
```

> kubectl patch -n monitoring prometheus-server --patch-file \<patch-file\>


```
data:
  alertmanager.yml: |
    global:
      resolve_timeout: 10m
      slack_api_url: Slack-URL
    receivers:
    - name: default-receiver
    - name: slack
      slack_configs:
      - channel: #development
        send_resolved: true
        title: '[{{.Status | toUpper}}] {{ .CommonLabels.alertname }}'
        text: |
          *Description:* {{ .CommonAnnotations.description }}
    route:
      group_interval: 1m
      group_wait: 10s
      receiver: slack
      repeat_interval: 5m
```

> kubectl patch cm prometheus-alertmanager -n monitoring --patch-file \<patch-file\>

> kubectl scale deployment nginx --replicas 0  

----- alert to slack ------

> kubectl scale deployment nginx --replicas 2






