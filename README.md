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