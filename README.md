## prometheus


- 여러 종류의 exporter들
    - cAdvisor : container metric를 수집. 
    - nodeExporter : node system 에 대한 metric를 수집. 
    - kube-state-metric : 각 object들에 대한 정보. 
    - application-metric : application 단에서 준비한 metric
    - Target 지정. 
        ```
        annotations:
            prometheus.io/scrape: "true"
            prometheus.io/port: "9113"
        ``` 

- PromQL
- Prometheus Config
    - job 추가. 
    - Rule 규칙 생성. 
    - Alert Rule 생성. 
        - Slack에 alert 보내기. 
- Prometheus Operator Setting
- Prometheus Operator Rules & Alert
- Prometheus Operator harbor
    - 외부 URL에 대한 Metirc 수집
- Prometheus Operator realenv
    - 실제 환경에서 사용되는 Operator

