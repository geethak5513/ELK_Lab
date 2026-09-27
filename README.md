# ELK Stack Manual Installation Guide (RPM)

A step-by-step guide to manually installing and configuring the **ELK Stack (v7.17.18)** and **Filebeat** on an RPM-based Linux system (e.g., RHEL, CentOS, Oracle Linux).

---

## 1. Prerequisites (Swap File & SSH Configuration)

Before installing the stack, configure your environment to prevent memory crashes and SSH timeouts.

### Add Swap Space (2GB)
This helps prevent memory-related crashes on smaller instances:
```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

### Prevent SSH Disconnections
Edit your local local SSH config (`~/.ssh/config`) and add the following lines to send a keep-alive packet every 60 seconds:
```text
Host *
  ServerAliveInterval 60
  ServerAliveCountMax 3
```

---

## 2. Elasticsearch Installation & Configuration

### Download and Install
```bash
# Download the Elasticsearch 7.17.18 RPM file directly
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.17.18-x86_64.rpm

# Install the RPM package
sudo rpm -ivh elasticsearch-7.17.18-x86_64.rpm
```

### Configure JVM Heap Size
Open the configuration file:
```bash
sudo nano /etc/elasticsearch/jvm.options
```
Update or add the following lines to limit memory usage:
```text
-Xms256m
-Xmx256m
```

### Configure Elasticsearch Network Settings
Open the main configuration file:
```bash
sudo nano /etc/elasticsearch/elasticsearch.yml
```
Append or modify the network and discovery configurations:
```yaml
network.host: localhost
http.port: 9200
discovery.type: single-node
```

### Enable, Start, and Test Service
```bash
# Enable and start the service
sudo systemctl enable elasticsearch
sudo systemctl start elasticsearch

# Test if Elasticsearch is responding
curl http://localhost:9200
```

---

## 3. Kibana Installation & Configuration

### Download and Install
```bash
# Download the Kibana 7.17.18 RPM file directly
wget https://artifacts.elastic.co/downloads/kibana/kibana-7.17.18-x86_64.rpm

# Install the RPM package
sudo rpm -ivh kibana-7.17.18-x86_64.rpm
```

### Configure Kibana Network Settings
Open the configuration file:
```bash
sudo nano /etc/kibana/kibana.yml
```
Modify the following lines to allow external connections and connect to Elasticsearch:
```yaml
server.host: "0.0.0.0"
elasticsearch.hosts: ["http://localhost:9200"]
```

### Network & Firewall Settings
If you are using a firewall, open port `5601`:
```bash
sudo firewall-cmd --permanent --add-port=5601/tcp
sudo firewall-cmd --reload
```
> ⚠️ **Note:** Make sure port `5601` is also open in your cloud infrastructure security list (e.g., your Oracle Cloud Infrastructure (OCI) VCN Security Lists / Network Security Groups).

### Start Service and Access Dashboard
```bash
# Start and Enable Kibana
sudo systemctl enable kibana
sudo systemctl start kibana
```
Open your web browser and navigate to:
```text
http://<your-instance-public-ip>:5601
```

---

## 4. Filebeat Installation & Configuration

### Download and Install
```bash
# Download the Filebeat 7.17.18 RPM file directly
wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.17.18-x86_64.rpm

# Install the RPM package
sudo rpm -ivh filebeat-7.17.18-x86_64.rpm
```

### Configure Filebeat
Open the configuration file:
```bash
sudo nano /etc/filebeat/filebeat.yml
```
Update the input and output blocks accordingly:

**Input Section:**
```yaml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /home/opc/app.log
```

**Output Section:**
```yaml
output.elasticsearch:
  hosts: ["http://localhost:9200"]
```

### Start and Verify Filebeat
```bash
# Start and Enable Filebeat
sudo systemctl enable filebeat
sudo systemctl start filebeat

# Verify real-time logs to ensure it starts cleanly
sudo journalctl -u filebeat -f
```

---

## 5. Verification & Testing

To test the entire pipeline end-to-end, create a dummy script to generate mock application logs.

### Create a Log Generator Script
Create a script named `log-generator.sh`:
```bash
nano log-generator.sh
```
Paste the following block into the file:
```bash
#!/bin/bash
while true
do
  echo "\$(date) - INFO - This is a test log entry" >> /home/opc/app.log
  sleep 5
done
```

### Run the Log Generator
```bash
# Make the script executable
chmod +x log-generator.sh

# Run the generator
./log-generator.sh
```

### Test Data Ingestion
Check if Elasticsearch is receiving the logs sent by Filebeat by running:
```bash
curl -X GET "localhost:9200/filebeat-*/_search?pretty"
```
















