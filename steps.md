

```
git clone https://github.com/gorkemozlu/kube-burner
export HARBOR_PULL_URL=harbor.local
export HARBOR_PUSH_URL=harbor.k8s.lab

export IMG1_ORG=k8s.gcr.io/kube-state-metrics/kube-state-metrics:v2.3.0
export IMG1=$HARBOR_PUSH_URL/$IMG1_ORG
export IMG1_PULL=$HARBOR_PULL_URL/$IMG1_ORG
sudo docker pull $IMG1_PULL
sudo docker tag $IMG1_PULL $IMG1
sudo docker push $IMG1

export IMG2_ORG=prom/prometheus
export IMG2=$HARBOR_PUSH_URL/docker.io/$IMG2_ORG
export IMG2_PULL=$HARBOR_PULL_URL/docker.io/$IMG2_ORG
sudo docker pull $IMG2_PULL
sudo docker tag $IMG2_PULL $IMG2
sudo docker push $IMG2

export IMG3_ORG=grafana/grafana:7.5.15
export IMG3=$HARBOR_PUSH_URL/docker.io/$IMG3_ORG
export IMG3_PULL=$HARBOR_PULL_URL/docker.io/$IMG3_ORG
sudo docker pull $IMG3_PULL
sudo docker tag $IMG3_PULL $IMG3
sudo docker push $IMG3

export IMG4_ORG=prom/node-exporter
export IMG4=$HARBOR_PUSH_URL/docker.io/$IMG4_ORG
export IMG4_PULL=$HARBOR_PULL_URL/docker.io/$IMG4_ORG
sudo docker pull $IMG4_PULL
sudo docker tag $IMG4_PULL $IMG4
sudo docker push $IMG4

export IMG5_ORG=docker.io/bitnami/elasticsearch:7.15.2
export IMG5=$HARBOR_PUSH_URL/$IMG5_ORG
export IMG5_PULL=$HARBOR_PULL_URL/$IMG5_ORG
sudo docker pull $IMG5_PULL
sudo docker tag $IMG5_PULL $IMG5
sudo docker push $IMG5

export IMG6_ORG=docker.io/bitnami/bitnami-shell:10-debian-10-r138
export IMG6=$HARBOR_PUSH_URL/$IMG6_ORG
export IMG6_PULL=$HARBOR_PULL_URL/$IMG6_ORG
sudo docker pull $IMG6_PULL
sudo docker tag $IMG6_PULL $IMG6
sudo docker push $IMG6

export IMG7_ORG=k8s.gcr.io/pause:3.1
export IMG7=$HARBOR_PUSH_URL/$IMG7_ORG
export IMG7_PULL=$HARBOR_PULL_URL/$IMG7_ORG
sudo docker pull $IMG7_PULL
sudo docker tag $IMG7_PULL $IMG7
sudo docker push $IMG7

sed -i -e "s~$IMG1_ORG~$IMG1~g" ./dependencies/kube-state-metrics/ksm.yaml
sed -i -e "s~$IMG2_ORG~$IMG2~g" ./dependencies/prometheus/deploy-prometheus.yaml
sed -i -e "s~$IMG3_ORG~$IMG3~g" ./dependencies/grafana/deploy-grafana.yaml
sed -i -e "s~$IMG4_ORG~$IMG4~g" ./dependencies/node-exporter/deploy-node-exporter.yaml
sed -i -e "s~$IMG5_ORG~$IMG5~g" ./dependencies/elasticsearch/deploy-elastic.yaml
sed -i -e "s~$IMG6_ORG~$IMG6~g" ./dependencies/elasticsearch/deploy-elastic.yaml
sed -i -e "s~$IMG7_ORG~$IMG7~g" ./examples/workloads/api-intensive/templates/deployment_patch_add_pod_2.yaml
sed -i -e "s~$IMG7_ORG~$IMG7~g" ./examples/workloads/api-intensive/templates/deployment.yaml      
```


```
wget https://github.com/cloud-bulldozer/kube-burner/releases/download/v0.15.4/kube-burner-0.15.4-Linux-x86_64.tar.gz
tar -xvf kube-burner-0.15.4-Linux-x86_64.tar.gz
sudo cp kube-burner /usr/local/bin/kube-burner
```


```
kubectl create ns monitoring
kubectl apply -f dependencies/elasticsearch
kubectl apply -f dependencies/node-exporter
kubectl apply -f dependencies/kube-state-metrics
kubectl apply -f dependencies/prometheus
kubectl apply -f dependencies/grafana
kubectl get svc,pods -n monitoring
#import dashboard examples/grafana-dashboards/api-and-etcd.json
export PROMETHEUS_URL=$(kubectl get svc prometheus-service -n monitoring -o jsonpath="{.status.loadBalancer.ingress[0].ip}")
export ELASTIC_URL=$(kubectl get svc elasticsearch -n monitoring -o jsonpath="{.status.loadBalancer.ingress[0].ip}")
export ELASTIC_URL_ORG=ELASTIC_URL
echo $PROMETHEUS_URL $ELASTIC_URL
#update examples/workloads/api-intensive/api-intensive.yml
sed -i -e "s~$ELASTIC_URL_ORG~$ELASTIC_URL~g" ./examples/workloads/api-intensive/api-intensive.yml
cd examples/workloads/api-intensive
kube-burner init -c api-intensive.yml -u http://$PROMETHEUS_URL -m ../../metrics-profiles/etcdapi.yml
```
