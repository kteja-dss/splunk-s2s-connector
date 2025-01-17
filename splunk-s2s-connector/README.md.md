# Confluent for Kubernetes Scenario Examples

This GitHub repository accompanies the official [Confluent for Kubernetes documentation](https://docs.confluent.io/operator/current/overview.html).

This repository contains scenario workflows to deploy and manage Confluent
on Kubernetes for various use cases.

## Prerequisites

The following prerequisites are assumed for each scenario workflow:

- A Kubernetes cluster - any CNCF conformant version
- Helm 3 installed on your local machine
- Kubectl installed on your local machine
- A namespace created in the Kubernetes cluster - `confluent`
- Kubectl configured to target the `confluent` namespace:
  ```
  kubectl config set-context --current --namespace=confluent
  ```
- This repo cloned to your workstation:
  ```
  git clone git@github.com:confluentinc/confluent-kubernetes-examples.git
  ```

# Next Steps

- Clone this repo ''
- Update the properties of the files ccloud-credentials.txt and ccloud-sr-credentials.txt
- Create secrets for cloud and connect

  ```
   kubectl create secret generic ccloud-credentials --from-file=plain.txt=ccloud-credentials.txt
  ```

  ```
   kubectl create secret generic ccloud-credentials --from-file=plain.txt=ccloud-sr-credentials.txt
  ```

- Change the [CONFLUENT_CLOUD_BOOTSTRAP_SERVER] and [SCHEMA_REGISTRY_URL] properties in the [connect.yaml] file with your boostratp server
- Apply the connect cluster CRD using the below command
  ```
  kubectl apply -f connect.yaml
  ```
- Setup the connector by sending a post request using the following command

  ```
  curl -X PUT \
  -H "Content-Type: application/json" \
  --data '{
    "connector.class": "io.confluent.connect.splunk.s2s.SplunkS2SSourceConnector",
    "tasks.max": "1",
    "splunk.s2s.port": "9997",
    "splunk.tcp.host": "18.116.200.213",
    "kafka.topic": "splunk-s2s-events",
    "key.converter": "org.apache.kafka.connect.storage.StringConverter",
    "value.converter": "org.apache.kafka.connect.json.JsonConverter",
    "key.converter.schemas.enable": "false",
    "value.converter.schemas.enable": "false",
    "confluent.topic.replication.factor": "1",
    "confluent.topic.bootstrap.servers": "<BOOTSTRAP_END_POINT>",
    "confluent.topic.security.protocol": "SASL_SSL",
    "confluent.topic.sasl.mechanism": "PLAIN",
    "confluent.topic.sasl.jaas.config": "org.apache.kafka.common.security.plain.PlainLoginModule required username=\"<BOOTSTRAP_SERVER_API_KEY\" password=\"<BOOTSTRAP_SERVER_API_SECRET\";",
  }' \
  http://localhost:8083/connectors/splunk-s2s-source/config | jq .

  ```

- Check the status of the connector and also check the data in the topic created on the Confluent Cloud
  ```
  http://localhost:8083/connectors/splunk-s2s-source/status
  ```
