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




