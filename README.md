
# kubernetes-efk

This repository contains the config to deploy an EFK stack on to an existing Kubernetes cluster. 

In the current state:

- Fluentd is deployed as a DaemonSet, and will send corresponding logs in `kubernetes-conf.yaml` to ElasticSearch.

- Elasticsearch is deployed as 2 StatefulSet and bind with PVC of AWS EBS.

- Kibana acts a stateless service that consumes data from ElasticSearch.

## Installation

- Setup `KUBECONFIG` in local so as to enable the auth to k8s cluster

- Run `auto/deploy-efk-on-k8s` to deploy the EFK stack.

- Forward kibana service to local

  ```
  kubectl port-forward -n eks svc/kibana 5601:5601
  ```

- Access the Kibana dashboard

  http://localhost:5601
