
### 오퍼레이터 환경에서 새로운 수집대상 설정 과정. 

1. 사용자가 Prometheus 를 helm upgrade로 설치.
2. 새로운 Target들에 대해서 secret 으로 정보를 보관. 
3. Prometheus에서는 해당 secret과 연동을 하게 됨. 
4. Prometheus Operator는 해당 Prometheus를 인지하고, Prometheus가 인지할 수 있는 정보를 secret에 다시 작성. 
5. Prometheus server에서는 해당 secret정보를 통해서 metric을 수집.

--------------

```
외부영역에 harbor container가 동작하고 있다고 가정. 
외부에서 harbor repo로 접근이 가능해야함 (인증서 설정 필요.)
--------
host pc 에서 192.168.1.92(harbor repo) 로 접근 가능하면 완료.
host pc 에서 다음과 같은 명령어로 metric이 수집 가능해야함
>> curl http://192.168.1.92:9090/metrics
```

외부의 harbor metric에 대한 수집 target을 Prometheus에 등록

```
#add-harbor.yaml
prometheus:
  prometheusSpec:
    additionalScrapeConfigs:
    - job_name: 'harbor'
      metrics_path: /metrics
      static_configs:
      - targets:
        - 192.168.1.92:9090
-----
> helm upgrade prometheus-stack edu/prometheus-stack \
    --set defaultRules.create="false" \
    --set alertmanager.enabled="false" \
    --set grafana.enabled="false" \
    --set prometheus.service.type="LoadBalancer" \
    --set prometheus.service.port="80" \
    --set prometheus.prometheusSpec.scrapeInterval="15s" \
    --set prometheus.prometheusSpec.evaluationInterval="15s" \
    --namespace=monitoring \
    -f ~/add-harbor.yaml
```

이렇게 되면, helm을 통해서 새로운 secret정보를 보관 (위의 2번과정)




