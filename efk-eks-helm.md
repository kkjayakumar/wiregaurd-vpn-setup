# EFK Stack Setup on EKS using Helm

This guide explains how to deploy the EFK (Elasticsearch, Fluent Bit, Kibana) stack on an AWS EKS cluster using Helm charts.

---

## Prerequisites

- AWS EKS cluster up and running
- `kubectl` configured for your cluster
- [Helm](https://helm.sh/) installed
- Sufficient IAM permissions for EBS and EKS

---

## 1. Create a StorageClass for Elasticsearch

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: efk-pv
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
volumeBindingMode: WaitForFirstConsumer
```

Apply it:

```sh
kubectl apply -f storageclass.yaml
```

---

## 2. Add Helm Repositories

```sh
helm repo add elastic https://helm.elastic.co
helm repo add fluent https://fluent.github.io/helm-charts
helm repo update
```

---

## 3. Deploy Elasticsearch

```sh
helm install elasticsearch \
  --set service.type=LoadBalancer \
  --set volumeClaimTemplate.storageClassName=efk-pv \
  --set persistence.labels.enabled=true \
  elastic/elasticsearch -n efk --create-namespace
```

Check pod status:

```sh
kubectl get pods -n efk -l app=elasticsearch-master -w
```

Retrieve the elastic user password:

```sh
kubectl get secrets -n efk elasticsearch-master-credentials -ojsonpath='{.data.password}' | base64 -d
```

---

## 4. Deploy Kibana

```sh
helm install kibana \
  --set service.type=LoadBalancer \
  elastic/kibana -n efk
```

Check pod status:

```sh
kubectl get pods -n efk -l release=kibana -w
```

---

## 5. Deploy Fluent Bit

Create a `fluentbit-values.yaml` file for custom configuration if needed.

Install Fluent Bit:

```sh
helm install fluent-bit fluent/fluent-bit -f fluentbit-values.yaml -n efk
```

---

## 6. Fluent Bit Example Config

Example `config.conf` for Fluent Bit:

```ini
[SERVICE]
    Daemon Off
    Flush 5
    Log_Level info
    Parsers_File /fluent-bit/etc/parsers.conf
    HTTP_Server On
    HTTP_Listen 0.0.0.0
    HTTP_Port 2020
    Health_Check On

[INPUT]
    Name tail
    Path /var/log/containers/*.log
    Exclude_Path /var/log/containers/fluent*
    Tag kube.*
    Parser docker_kubernetes
    DB /var/log/flb_kube.db
    Mem_Buf_Limit 100MB
    Skip_Long_Lines On
    Refresh_Interval 10

[FILTER]
    Name kubernetes
    Match kube.*
    Merge_Log On
    Keep_Log Off
    K8S-Logging.Parser On
    K8S-Logging.Exclude Off
    Buffer_Size 256KB

[OUTPUT]
    Name es
    Match kube.*
    Host elasticsearch-master.efk.svc.cluster.local
    Port 9200
    HTTP_User elastic
    HTTP_Passwd <ELASTIC_PASSWORD>
    TLS On
    TLS.Verify Off
    Logstash_Format On
    Replace_Dots On
    Retry_Limit False
    Suppress_Type_Name On

[PARSER]
    Name docker_kubernetes
    Format json
    Time_Key time
    Time_Format %Y-%m-%dT%H:%M:%S.%L
    Time_Keep On
    Decode_Field_As escaped_utf8 log
```

---

## 7. Access Kibana

Get the Kibana service's external URL:

```sh
kubectl get svc -n efk
```

Open the Kibana URL in your browser and log in with the `elastic` user and password retrieved earlier.

---

## References

- [Elastic Helm Charts](https://www.elastic.co/guide/en/elastic-stack-get-started/current/get-started-elastic-stack.html)
- [Fluent Bit Helm Chart](https://github.com/fluent/helm-charts/tree/main/charts/fluent-bit)
- [Deploying EFK Stack on Kubernetes using Helm](https://medium.com/@navae.izidbiha/deploying-efk-stack-on-kubernetes-cluster-using-helm-charts-a9147e9a71e0)

---
