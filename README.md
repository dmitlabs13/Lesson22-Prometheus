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






ставим экспортер на второй сервер  

```
sudo apt install prometheus-node-exporter -y
```

делаем конфиг 
```
global:
  scrape_interval: 15s # Интервал, по которому идет обращение к таргетам

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node'
    file_sd_configs:
      - files:
        - /etc/prometheus/target.json
```

создаем файл /etc/prometheus/target.json
```

  {
    "targets": ["192.168.50.227:9100", "192.168.1.11:9100"],
    "labels": {
      "env": "production",
      "datacenter": "msk"
    }
  }
]

```

перезагружаем сервис и проверяем результат root@lp-ubn1:/tmp# sudo systemctl restart prometheus  
<img width="1456" height="590" alt="image" src="https://github.com/user-attachments/assets/ccf4229d-cd26-4443-bb16-eb4a4c7ef2cd" />  


ставим grafana
```
# Установка grafana
sudo apt update && sudo apt upgrade -y
sudo apt install -y apt-transport-https software-properties-common wget
# так как блокирется установка через apt, ставим через deb пакет и перекинем на ВМ
wget https://dl.grafana.com/grafana/release/13.1.0/grafana_13.1.0_28013217238_linux_amd64.deb
sudo systemctl daemon-reexec
sudo systemctl enable grafana-server
sudo systemctl start grafana-server

```

смотрим http://192.168.50.238:3000/ работает  

<img width="663" height="576" alt="image" src="https://github.com/user-attachments/assets/b5b51913-0e1d-4697-ae9f-f0799ab1f8f5" />



Настроил дашборд  
<img width="1375" height="716" alt="image" src="https://github.com/user-attachments/assets/3c91a42c-bb98-4897-a8fe-2eb5e9cea00e" />







