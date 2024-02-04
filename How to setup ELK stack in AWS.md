
<h3> <b>Step 1 :</b></h3> Creating instance in AWS with below configuration 

	Required Configuration
			* 8 CPU 
			* 32GB Memory 
			* 80 GB HDD 
			* Create Security Network Policy and allow port 9200,5601,5044
<h3> <b>Step 2 :</b></h3> 
Install Required package :

 ```` bash
	sudo apt update
	sudo apt install curl 
	sudo apt install wget 
	sudo apt install gnupg2
	sudo apt install openjdk-11-jdk
	sudo apt install apt-transport-https 
````

Add Repository :

````bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch --no-check-certificate | sudo apt-key add -

echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-7.x.list
````


&nbsp;

Update the Linux packages post adding the repository :

```` bash
	sudo apt update 
````


&nbsp;
&nbsp;

<h3><span style="color: yellow"> Elasticsearch </span></h3>
 Install Elasticsearch package:

 ``` bash
	 sudo apt install elasticsearch
```
 
Configure elasticsearch configuration file using #nano editor 

     Path : sudo nano /etc/elasticsearch/elasticsearch.yml

``` yml
	Remove # tag before the line 
	network.host:localhost
	port:9200
```

Enable and Start the Elasticsearch:

```bash 
	sudo systemctl enable elasticsearch.service
	sudo systemctl start elasticsearch.service
```

To verify elasticsearch:

``` bash
	curl -XGET 'http://localhost:9200'
	curl -XGET 'localhost:9200/_cluster/health?pretty'	
```

 <h3><span style="color: yellow"> Kibana </span></h3>
 Install Kibana package:

 ``` bash
	 sudo apt install kibana
```
 
Configure kibana configuration file using #nano editor 

     Path : sudo nano /etc/kibana/kibana.yml

``` yml
	Remove # tag before the line 
	ip.address:'0.0.0.0' # it will accept any IP address 
	port:5601
	elasticsearch "localhost:9200"
```

Enable and Start the kibana:

```bash 
	sudo systemctl enable kibana.service
	sudo systemctl start kibana.service
```

To verify kibana:

Try to open the kibana url on the browser 

``` html
	http://<ipaddress:5601>
```

 <h3><span style="color: yellow"> Logstash </span></h3>
  Install Logstash package:

 ``` bash
	 sudo apt install logstash
```

#Create a configuration file called <b>2-beats-input.conf </b> where you will set up your Filebeat input:

	  Path : sudo nano /etc/logstash/conf.d/2-beats-input.conf

Insert below lines on above mentioned #path 

```c
input {
  beats {
    port => 5044
  }
}

output {
  stdout {}
}
```

#Next, create a configuration file called <b> 2-elasticsearch-output.conf </b>

	  Path : sudo nano /etc/logstash/conf.d/2-elasticsearch-output.conf

Insert below lines on above mentioned #path.Essentially, this output configures Logstash to store the Beats data in Elasticsearch, which is running at localhost:9200, in an index named after the Beat used. The Beat used in this tutorial is Filebeat

```c
output {
  if [@metadata][pipeline] {
	elasticsearch {
  	hosts => ["localhost:9200"]
  	manage_template => false
  	index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
  	pipeline => "%{[@metadata][pipeline]}"
	}
  } else {
	elasticsearch {
  	hosts => ["localhost:9200"]
  	manage_template => false
  	index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
	}
  }
}
```

Enable and Start the logstash:

```bash 
	sudo systemctl enable logstash
	sudo systemctl start logstash
```

Start the logstash with pipeline configuration:

``` bash
	sudo /usr/share/logstash/bin/logstash -f 2-elasticsearch-output.conf
```

Test your logstash configuration with below command:

```bash 
	sudo -u logstash /usr/share/logstash/bin/logstash --path.settings /etc/logstash -t
```


 <h3><span style="color: yellow"> Apache 2  ( Linux Web Server )</span></h3>
  Install Apache 2 package: 

 ``` bash
	 sudo apt install apache2
```


 <h3><span style="color: yellow"> Filebeat</span></h3>
 Download the filebeat:

 ``` bash
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.14.0-amd64.deb sudo dpkg -i filebeat-7.14.0-amd64.deb
```

Configure filebeat configuration file using #nano editor 

     Path : sudo nano /etc/filebeat/filebeat.yml 

``` yml
	Remove # tag before the line 
	output.logstash
	hosts:['localhost:5044']
	# Change the value to true to enable the input configuration
	enabled: true
	# Add the apache2 web server logs path 
	- /var/log/apache2/access.log
	# Set to True to enable the configuration reloading
	reload.enabled:true
```


Enable the Filebeat module:

```bash
	sudo filebeat modules enable system
```

Load the index Template using below command:

```bash 
sudo filebeat setup --index-management -E output.logstash.enabled=false -E 'output.elasticsearch.hosts=["localhost:9200"]'
```

Enable and Start the Filebeat:

```bash 
	sudo systemctl enable filebeat
	sudo systemctl start filebeat
```


Post Configuration go to kibana and set the index 

- Go to Index Management and 
- Set the index filebeat-*
- Validate the indexing by going to discover tab 

To Verify :

Filebeat index :

```bash
	curl -XGET 'http://localhost:9200/filebeat-*/_search?pretty'
```

Indices

```bash
	curl -XGET 'http://localhost:9200/_cat/indices'
```

Filebeat logstash health

```bash
	curl http://localhost:9200/filebeat-*/_count?pretty
```

Log Flow 

```bash
	journalctl -u filebeat.service
```


