# boot-elk
The repo demonstrates how to create a simple springboot application, to run on EC2 instance and hwo to configure Kibana.


## Features
-Create an EC2 instance.
-Install Java.
-Install Elasticseach.
-Install Logstash.
-Install Kibana.
-Build application and run on EC2

## Create an EC2 instance
Ubuntu Server 20.04 LTS (HVM)
## Install Java
```
sudo apt-get install default-jre
```
## Install Elastic search 
```
-wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```
## Install Apt transport HTTPS
```
sudo apt-get install apt-transport-https
```
```echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
sudo apt-get update && sudo apt-get install elasticsearch
```
- Updtae the “/etc/elasticsearch/elasticsearch.yml” by uncommenting below lines:
    ```
    node.name: node-1
    network.host: "localhost"
    http.port: 9200
    cluster.initial_master_nodes: ["node-1"]
    ```
- Edit the “/etc/elasticsearch/jvm.options” file with below changes
 ```
 -Xms4g
```
```sudo su - 
service elasticsearch start
```
```
curl http://IPAddress:9200
```
## Installing Logstash
```
sudo apt-get install logstash
```
- Create a sample file at the location “/home” named as “test_log.log” and include just a few lines for testing. Some content like,
  
  ```
  INFO somelog
  ERR somelog2
  ERR somelog3
  ```
-Create a file with “sudo nano /etc/logstash/conf.d/apache-01.conf”
```
input {
  file {
    type => "java"
    path => "/home/test_log.log"
    codec => multiline {
      pattern => "^%{YEAR}-%{MONTHNUM}-%{MONTHDAY} %{TIME}.*"
      negate => "true"
      what => "previous"
    }
  }
}
 
filter {
  #If log line contains tab character followed by 'at' then we will tag that entry as stacktrace
  if [message] =~ "\tat" {
    grok {
      match => ["message", "^(\tat)"]
      add_tag => ["stacktrace"]
    }
  }
 
}
 
output {
   
  stdout {
    codec => rubydebug
  }
 
  # Sending properly parsed log events to elasticsearch
  elasticsearch {
    hosts => ["localhost:9200"]
  }
}
```
NOTE: Change the log file permission to 755
```
sudo service logstash start
```
- To check 
```
sudo curl -XGET 'localhost:9200/_cat/indices?v&pretty'
```

##Install Kibana
```
sudo apt-get install kibana
```
-Edit the file "/etc/kibana/kibana.yml"
```
server.port: 5601
server.host: "0.0.0.0"
elasticsearch.hosts: ["http://localhost:9200"]
```
- Start Kibana
```
sudo service kibana start
```
- Access Kibana
```
http://<IP>:5601
```

## Running the springboot application 
- Do git clone to repo on the directory
- Run "mvn clean install -DskipTests"
- Run the Jar " java -jar spring-boot-elk-0.0.1-SNAPSHOT.jar > /home/ubuntu/jar/log/log1.log"

Note : Update the apllication properites log file path to "/home/test_log.log"



 
