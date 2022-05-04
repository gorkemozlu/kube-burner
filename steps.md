

```
git clone
pull push images
#update images
#update api-intensive.yml
wget https://github.com/cloud-bulldozer/kube-burner/releases/download/v0.15.4/kube-burner-0.15.4-Linux-x86_64.tar.gz
tar -xvf kube-burner-0.15.4-Linux-x86_64.tar.gz
sudo cp kube-burner /usr/local/bin/kube-burner
```


```
kubectl apply -f dependencies/node-exporter
kubectl apply -f dependencies/kube-state-metrics
kubectl apply -f dependencies/prometheus
kubectl apply -f dependencies/grafana
#import dashboard
kubectl apply -f dependencies/elasticsearch
cd examples/workloads/api-intensive
kube-burner init -c api-intensive.yml -u http://prometheus.dorn.gorke.ml -m ../../metrics-profiles/etcdapi.yml      
```