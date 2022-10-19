### Coletando Metricas do Node

Ref. 
https://prometheus.io/docs/guides/node-exporter/
https://jaanhio.me/blog/linux-node-exporter-setup/

```bash
curl -LO  https://github.com/prometheus/node_exporter/releases/download/v1.4.0/node_exporter-1.4.0.linux-amd64.tar.gz --output-dir /tmp
tar -xf /tmp/node_exporter-1.4.0.linux-amd64.tar.gz -C /tmp
sudo mv /tmp/node_exporter-1.4.0.linux-amd64/node_exporter  /usr/local/bin
```
```bash
sudo useradd -m node_exporter
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```
#### Criando serviço  do node_exporter

sudo vim /etc/systemd/system/node_exporter.service

```ini
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
```bash
sudo systemctl start node_exporter
sudo systemctl enable node_exporter
sudo systemctl status node_exporter
```

#### Adicione o job no bloco do scrape_configs no arquivo promethues.yml no servidor do prometheus

`sudo vim /etc/prometheus/prometheus.yml`

```yml
- job_name: node
  static_configs:
  - targets: ['<IP-DO-ALVO>:9100', '<IP-DO-ALVO-02>:9100' ]
```

```sh
sudo systemctl restart prometheus
sudo systemctl status prometheus
```

### Dashboard para visualizar metricas do node no grafana

https://grafana.com/grafana/dashboards/1860-node-exporter-full/
