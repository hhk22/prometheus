
### cAdvisor

prefix = container_

kubelet이 cAdvisor를 통해서 container에 대한 metric을 수집. 

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

/sys
/proc

부하주기

> stress --vm 5 --vm-bytes 10M --timeout 150s

metirc: `node_memory_Active_bytes{node="w2-k8s"}`

