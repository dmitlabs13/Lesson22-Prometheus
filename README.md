# Lesson22-Prometheus

## установка серверной части на vst-ubn1

```
sudo apt update && sudo apt upgrade -y
sudo useradd --no-create-home --shell /bin/false prometheus
sudo mkdir /etc/prometheus /var/lib/prometheus
sudo chown prometheus:prometheus /etc/prometheus /var/lib/prometheus
cd /tmp
wget https://github.com/prometheus/prometheus/releases/download/v3.12.0/prometheus-3.12.0.linux-amd64.tar.gz
tar xvf prometheus-3.12.0.linux-amd64.tar.gz
sudo cp prometheus-3.12.0.linux-amd64/prometheus /usr/local/bin/
sudo cp prometheus-3.12.0.linux-amd64/promtool /usr/local/bin/
sudo chown prometheus:prometheus /usr/local/bin/prometheus /usr/local/bin/promtool
sudo cp -r prometheus-3.12.0.linux-amd64/consoles /etc/prometheus/
sudo cp -r prometheus-3.12.0.linux-amd64/console_libraries /etc/prometheus/
sudo chown -R prometheus:prometheus /etc/prometheus/consoles /etc/prometheus/console_libraries
sudo nano /etc/prometheus/prometheus.yml

global:
  scrape_interval: 15s # Интервал, по которому идет обращение к таргетам

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

sudo nano /etc/systemd/system/prometheus.service
``
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
    --web.listen-address=0.0.0.0:9090

[Install]
WantedBy=multi-user.target
``
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl start prometheus

```

Заходим на http://192.168.50.238:9090/metrics  
<img width="730" height="512" alt="image" src="https://github.com/user-attachments/assets/4e13c24c-c790-4182-9dae-3dc0e164ba6c" />


Заходим на   http://192.168.50.238:9090/
<img width="629" height="511" alt="image" src="https://github.com/user-attachments/assets/b5dd94f6-53ff-4956-a960-22594a875a73" />






ставим экспортер

```
sudo apt install prometheus-node-exporter -y
```
