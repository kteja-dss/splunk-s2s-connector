# Confluent for Kubernetes Scenario Examples

More info can be found on [Confluent for Kubernetes documentation](https://docs.confluent.io/operator/current/overview.html).

This repository contains simple schenario and can be customised with respect to the prod requirements after testing

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
- Set up the Helm Chart:

  ```
  helm repo add confluentinc https://packages.confluent.io/helm
  ```

  Install Confluent For Kubernetes using Helm:

  ```
  helm upgrade --install operator confluentinc/confluent-for-kubernetes -n confluent
  ```

  Check that the Confluent For Kubernetes pod comes up and is running:

  ```
  kubectl get pods -n confluent
  ```

- This repo cloned to your workstation:
  ```
  git clone git@github.com:confluentinc/confluent-kubernetes-examples.git
  ```

# Next Steps

- Clone this repo "https://github.com/kteja-dss/splunk-s2s-connector.git"
- Open terminal in the repo directory
- Update the credentials in the files ccloud-credentials.txt and ccloud-sr-credentials.txt
- Create secrets for cloud, sr and connect license

  ```
   kubectl create secret generic ccloud-credentials --from-file=plain.txt=ccloud-credentials.txt
  ```

  ```
  kubectl create secret generic ccloud-sr-credentials --from-file=basic.txt=ccloud-sr-credentials.txt
  ```
  
Replace your license key with <YOUR_ENTERPRISE_LICENSE_KEY> in the license.txt and store it as a secret
    ```
    kubectl create secret generic conn-license \
      --from-file=connector-license.txt=./license.txt \
      --namespace confluent
    ```

- Change the [BOOTSTRAP_URL] and [SCHEMA_REGISTRY_URL] properties in the [connect.yaml] file with your boostratp server
- Apply the connect cluster CRD using the below command

  ```
  kubectl apply -f connect.yaml
  ```

- Change the configs like bootstra [<BOOTSTRAP_URL>] [BOOTSTRAP_KAFKA_API_KEY] [BOOTSTRAP_KAFKA_API_SECRET] and apply the connector CRD using the below command
  ```
  kubectl apply -f connector.yaml
  ```
  OR
- Portforward the connect cluster to send the api request to create the connector
  ```
  kubectl port-forward connect-0 8083
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
    "splunk.s2s.compression.enable": "true",
    "splunk.s2s.ack.enable": "true",
    "confluent.topic.sasl.jaas.config": "org.apache.kafka.common.security.plain.PlainLoginModule required username=\"<BOOTSTRAP_SERVER_API_KEY\" password=\"<BOOTSTRAP_SERVER_API_SECRET\";",
  }' \
  http://localhost:8083/connectors/splunk-s2s-source/config | jq .

  ```

- Check the status of the connector and also check the data in the topic created on the Confluent Cloud

  ```
  http://localhost:8083/connectors/splunk-s2s-source/status
  ```

- Create the loadbalance service for all our connect clusters
  ```
  kubectl apply -f connect-service.yaml
  ```
- Port forward the loadbalancer

```
kubectl port-forward svc/splunk-s2s-kafka-connect-lb 9997:9997
```
## For Heavy forwarder

- Download the splunk instance and configure your stanza to forward the traffic to the loadbalancer

```
https://www.splunk.com/en_us/download.html
```

- Configure the the stanzas accordingly, for a reference you can see those files in this repo
  ``inputs.conf `` and ``outputs.conf``
  

## For Universal forwarder

- Run Splunk forwarder docker image
  ```
  docker run -d -p 9998:9997 -e "SPLUNK_START_ARGS=--accept-license" -e "SPLUNK_PASSWORD=password" --name splunk-uf splunk/universalforwarder:9.0.0
  ```
- Create a log file `splunk-s2s-test.log` and add following events to it
  ```
  log event 1
  log event 2
  log event 3
  ```
- Copy the logfile and mount it with the docker container
  ```
  docker cp splunk-s2s-test.log splunk-uf:/opt/splunkforwarder/splunk-s2s-test.log
  ```
- Configure the UF to monitor the splunk-s2s-test.log file:
  ```
  docker exec -it splunk-uf sudo ./bin/splunk add monitor -source /opt/splunkforwarder/splunk-s2s-test.log -auth admin:password
  ```
- Configure the UF to connect to Splunk S2S Source connector:

  - For Mac/Windows systems:

    ```
    docker exec -it splunk-uf sudo ./bin/splunk add forward-server host.docker.internal:9997
    ```

  - For Linux systems:
    ```
    docker exec -it splunk-uf sudo ./bin/splunk add forward-server localhost:9997
    ```
