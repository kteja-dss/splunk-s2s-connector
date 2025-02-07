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
    docker exec -it splunk-uf sudo ./bin/splunk add forward-server 172.17.0.1:9997
    ```
