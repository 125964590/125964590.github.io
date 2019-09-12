- ## Install Filebeat
  This version 6.3.0 but my aliyun is 6.2.7
  - rpm
  ```
  curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-6.3.1-x86_64.rpm
  sudo rpm -vi filebeat-6.3.1-x86_64.rpm
  ```
  - docker
  ```
  docker pull docker.elastic.co/beats/filebeat:6.3.0
  ```

  ## Start Filebeat
  - rpm
  ```
  sudo service filebeat start
  ```
  - docker
  ```
  docker run docker.elastic.co/beats/filebeat:6.3.0
  ```

  ## Filebeat Directory layout
  - [Directory layout](https://www.elastic.co/guide/en/beats/filebeat/current/directory-layout.html)

- https://www.elastic.co/guide/en/beats/filebeat/current/directory-layout.html)