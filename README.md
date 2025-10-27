# Install Prometheus and Grafana on Ubuntu 24.04 LTS

## Step #1:Creating Prometheus System Users and Directory
- Create a system user for Prometheus using below commnds:
```sh
sudo useradd --no-create-home --shell /bin/false prometheus
```
- Create the directories in which we will be storing our configuration files and libraries:
```sh
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
```
- Set the ownership of the /var/lib/prometheus directory with below command:
```sh
sudo chown prometheus:prometheus /var/lib/prometheus
```

## Step #2:Download Prometheus Binary File
- You need to inside /tmp:
```sh
cd /tmp/
```
- Download the Prometheus setup using wget
```sh
wget https://github.com/prometheus/prometheus/releases/download/v2.46.0/prometheus-2.46.0.linux-amd64.tar.gz
```
- Extract the files using tar:
```sh
tar -xvf prometheus-2.46.0.linux-amd64.tar.gz
```
- Move the configuration file and set the owner to the prometheus user:
```sh
cd prometheus-2.46.0.linux-amd64
sudo mv console* /etc/prometheus
sudo mv prometheus.yml /etc/prometheus
sudo chown -R prometheus:prometheus /etc/prometheus
```
- Move the binaries and set the owner:
```sh
sudo mv prometheus /usr/local/bin/
sudo chown prometheus:prometheus /usr/local/bin/prometheus
```

## Step #3:Prometheus configuration file
- verify if it present and should look like below and modify it as per your requirement.
```sh
sudo nano /etc/prometheus/prometheus.yml
```

## Step #4:Creating Prometheus Systemd file
- Create the service file using below command:
```sh
sudo nano /etc/systemd/system/prometheus.service
```
```sh
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```
- Reload systemd:
```sh
sudo systemctl daemon-reload
```
- Start, Enable, Status Prometheus service:
```sh
sudo systemctl start prometheus
sudo systemctl enable prometheus
sudo systemctl status prometheus
```

## Step #5:Accessing Prometheus in Browser
- Use below command to enable prometheus service in firewall 
```sh
sudo ufw allow 9090/tcp
```
- Now Prometheus service is ready to run and we can access it from any web browser.
```sh
http://server-IP-or-Hostname:9090
```

## Step #6:Download Node Exporter
- using `cd /tmp`
```sh
cd /tmp
```
- Now lets run the copied URL with wget command
```sh
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
```
- Unzip the downloaded
```sh
sudo tar xvfz node_exporter-*.*-amd64.tar.gz
```
- Move the binary file of node exporter to /usr/local/bin location
```sh
sudo mv node_exporter-*.*-amd64/node_exporter /usr/local/bin/
```
- Create a node_exporter user to run the node exporter service
```sh
sudo useradd -rs /bin/false node_exporter
```

## Step #7:Creating Node Exporter Systemd service
- Create a node_exporter service file in the /etc/systemd/system directory
```sh
sudo nano /etc/systemd/system/node_exporter.service
```
```sh
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```
- Now lets start and enable the node_exporter service using below commands
```sh
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter
sudo systemctl status node_exporter
```

## Step #8:Configure the Node Exporter as a Prometheus target
- So go to etc/prometheus and open prometheus.yml
```sh
sudo nano /etc/prometheus/prometheus.yml
```
- Add new, `not change existing`
```sh
- job_name: 'Node_Exporter'
    scrape_interval: 5s
    static_configs:
      - targets: ['<Server_IP_of_Node_Exporter_Machine>:9100']
```
- Now restart the Prometheus Service
```sh
sudo systemctl restart prometheus
```
- Hit the URL in your web browser to check weather our target is successfully scraped by Prometheus or not