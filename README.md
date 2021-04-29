
# kubernetes-efk

This repository contains the config to deploy an EFK stack on to an existing Kubernetes cluster. 

## Architecture

![efk stack](./docs/efk-stack.png)

In the current state:

- Fluentd is deployed as a DaemonSet, and will send corresponding logs defined in `kubernetes-conf.yaml` to ElasticSearch.

- Elasticsearch is deployed as 2 StatefulSet and bind with PVC of AWS EBS.

- Kibana acts a stateless service that consumes data from ElasticSearch.

## Installation

- Setup `KUBECONFIG` in local so as to enable the auth to k8s cluster

- Run `auto/deploy-es` to deploy elasticsearch

- Initialize es credentials

  Once finished es deployment, run the following CMD to initialize es user credentials for any services that need talk to es

  ```
  $ kubectl exec -it $(kubectl get pods -n efk | grep es-cluster | sed -n 1p | awk '{print $1}') -n efk -- bin/elasticsearch-setup-passwords auto
  Changed password for user apm_system
  PASSWORD apm_system = xxxxxxxxxxxxxxxxx

  Changed password for user kibana
  PASSWORD kibana = xxxxxxxxxxxxxxx

  Changed password for user logstash_system
  PASSWORD logstash_system = xxxxxxxxxxxxxxxx

  Changed password for user beats_system
  PASSWORD beats_system = xxxxxxxxxxxxxxx

  Changed password for user remote_monitoring_user
  PASSWORD remote_monitoring_user = xxxxxxxxxxxxxxxx

  Changed password for user elastic
  PASSWORD elastic = xxxxxxxxxxxxxxxxxxxxxx
  ```

- Create es user/password via `secret`

  ```
  kubectl create secret -n efk generic es-secret \
    --from-literal=user=elastic \
    --from-literal=password=xxxxxxxxxxxxxxxxxxxxxx
  ```

- Run `auto/deploy-fluentd` to deploy fluentd

- Create es user/password for kibana via `secret`

  ```
  kubectl create secret -n efk generic kibana-secret \
    --from-literal=user=kibana \
    --from-literal=password=xxxxxxxxxxxxxxx
  ```

- Run `auto/deploy-kibana` to deploy kibana


- Forward kibana service to local

  ```
  kubectl port-forward -n eks svc/kibana 5601:5601
  ```

- Access the Kibana dashboard via elastic/xxxxxxxxxxxxxxx

  http://localhost:5601

- Checking anything happened from logs

  ```
  kubectl logs -n efk [statefulset/es-cluster | ds/fluentd | deployment/kibana] -f
  ```
