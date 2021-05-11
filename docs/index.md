<head>
  {% seo %}
 </head>
# Prometheus Kurulumu

## Kullanıcıları Oluşturulur.

```sh
sudo useradd --no-create-home --shell /bin/false prometheus
```

## Dizinler Oluşturulur.
```sh
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
```

## İzinler Verilir
```sh
sudo chown prometheus:prometheus /etc/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus
```
## Prometheus (2.26 sürümü ) İndirilir.
[Prometheus](files/prometheus-2.26.0.linux-amd64.tar.gz) Dosyasını indirin.

Alternatif
```sh
curl -LO https://github.com/prometheus/prometheus/releases/download/v2.26.0/prometheus-2.26.0.linux-amd64.tar.gz
```
## Arşivden Çıkartılır.
```sh
tar xvf prometheus-2.26.0.linux-amd64.tar.gz
```
## Dosya Kopyalanır.
```sh
sudo cp prometheus-2.26.0.linux-amd64/prometheus /usr/local/bin/
sudo cp prometheus-2.26.0.linux-amd64/promtool /usr/local/bin/
```

## Kopyalanan dosyanın izin ve grupları değiştirilir.
```sh
sudo chown -R prometheus:prometheus /etc/prometheus/consoles
sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries
```
##  Prometheus'u Yapılandırma
```sh
sudo vi /etc/prometheus/prometheus.yml
```
```yml
# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
    - targets: ['localhost:9010']

  - job_name: 'mongodb-exporter'
    static_configs:
    - targets: ['localhost:9011']
```


## Yapılandırma Dosyasının İzinleri

```sh
sudo chown prometheus:prometheus /etc/prometheus/prometheus.yml
```

## Servis Haline Getirilmesi
```sh
vi /etc/systemd/system/prometheus.service
```

```service
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
    --web.console.libraries=/etc/prometheus/console_libraries \
    --web.listen-address=:9010  

[Install]
WantedBy=multi-user.target
```
```sh
sudo systemctl daemon-reload &&
sudo systemctl enable prometheus &&
sudo systemctl start prometheus &&
sudo systemctl status prometheus &&
```

# Exporter Kurulumu

## MONGO_URI Değişkeni Atanır.
```sh
export MONGO_URI=mongodb://mongodb_exporter:password@mongo1-node:27017,mongo2-node:27017,mongo3-node:27017/?authSource=admin
```
:warning: hosts un altına tanımlamaları unutmayınız!
***Örnek***
```sh
#cat /etc/hosts
192.168.1.2 mongo1-node
193.168.1.3 mongo2-node
194.168.1.4 mongo3-node
```
## MongoDB de Kullanıcının Oluşturulması

```json
## mongodb'ye bağlanılır.
##> Burası güncellenecektir.
use admin

db.createRole({
    role: "explainRole",
    privileges: [{
        resource: {
            db: "",
            collection: ""
            },
        actions: [
            "listIndexes",
            "listCollections",
            "dbStats",
            "dbHash",
            "collStats",
            "find"
            ]
        }],
    roles:[]
})

db.getSiblingDB("admin").createUser({
   user: "mongodb_exporter",
   pwd: "password",
   roles: [
      { role: "explainRole", db: "admin" },
      { role: "clusterMonitor", db: "admin" },
      { role: "read", db: "local" }
   ]
})
```

## Exporter Dosyasının Kurulumu
```sh
sudo git clone https://github.com/erelbi/prometheus-grafana-mongo-exporter.git
cd prometheus-grafana-mongo-exporter/
sudo pip3 install -r requirements.txt
sudo mkdir -p  /opt/prometheus/mongodb-exporter
sudo cp -p -r  exporter/exporter.py /opt/prometheus/mongodb-exporter/
```
## İzinler 

```sh
sudo chown prometheus:prometheus -R /opt/prometheus
sudo chmod +x /opt/prometheus/mongodb-exporter/exporter.py
```

## Exporter Servis Haline Getirilmesi
```sh
vi /etc/systemd/system/mongo-exporter.service
```

```service
[Unit]
Description=MongoDB Replica set prometheus exporter
After=network.target

[Service]
User=prometheus
Group=prometheus
WorkingDirectory=/opt/prometheus/mongodb-exporter/
ExecStart=/opt/prometheus/mongodb-exporter/exporter.py
Type=simple

[Install]
WantedBy=multi-user.target
```
```sh
sudo systemctl daemon-reload &&
sudo systemctl enable mongo-exporter &&
sudo systemctl start mongo-exporter &&
sudo systemctl status mongo-exporter &&
```
# Grafana Kurulumu

```sh
sudo vi /etc/yum.repos.d/grafana.repo 
```
## Repo ( RedHat )
```repo
[grafana]
name=grafana
baseurl=https://packages.grafana.com/oss/rpm
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://packages.grafana.com/gpg.key
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
```
```sh
sudo yum update 
sudo yum install grafana 
```
```sh
sudo systemctl daemon-reload
sudo systemctl start grafana-server
sudo systemctl status grafana-server
```










