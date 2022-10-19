
### Coletando métricas do Nginx 

#### Instale o Nginx 
```
sudo apt install nginx -y
```

Ref.
1. https://igunawan.com/how-to-monitor-basic-metrics-from-nginx-with-prometheus-and-grafana/
2. https://github.com/nginxinc/nginx-prometheus-exporter
3. https://www.observability.blog/nginx-monitoring-with-prometheus/

#### Configuração e ativação do module stup_status do Nginx

Adicione a configuração abaixo no /etc/nginx/nginx.conf no bloco http{}

```nginx
server {
        listen localhost:81;
        location /metrics {
                stub_status on;
                allow 127.0.0.1;
                deny all;

        }
}

```
Reinicie o ngnix

```bash
sudo systemctl restart  nginx 
```
As métricas podem ser vistas no localhost usando `curl http://localhost:81/metrics` .
```output
Active connections: 2 
server accepts handled requests
 23 23 64 

```
#### Instalação e configuração do exporter do nginx para o prometheus

```sh
curl -LO  https://github.com/nginxinc/nginx-prometheus-exporter/releases/download/v0.11.0/nginx-prometheus-exporter_0.11.0_linux_amd64.tar.gz --output-dir /tmp
tar -xf /tmp/nginx-prometheus-exporter_0.11.0_linux_amd64.tar.gz -C /tmp
sudo mv /tmp/nginx-prometheus-exporter /usr/local/bin
```
Crie um usuário de serviço par o nginx exporter 

```sh
sudo useradd -m nginx_exporter
sudo chown nginx_exporter:nginx_exporter /usr/local/bin/nginx-prometheus-exporter

```

```sh
sudo vim  /etc/systemd/system/nginx_prometheus_exporter.service

```

```ini	
[Unit]
Description=NGINX Prometheus Exporter
After=network.target

[Service]
Type=simple
User=nginx_exporter
Group=nginx_exporter
ExecStart=/usr/local/bin/nginx-prometheus-exporter \
    --web.listen-address=0.0.0.0:9003 \
    --nginx.scrape-uri http://127.0.0.1:81/metrics

SyslogIdentifier=nginx_prometheus_exporter
Restart=always

[Install]
WantedBy=multi-user.target

```
#### Habilitando o serviço do nginx exporter

```sh
sudo systemctl start nginx_prometheus_exporter
sudo systemctl enable nginx_prometheus_exporter
sudo systemctl status nginx_prometheus_exporter
```
As métricas disponíveis pelo exporter podem ser vistas no localhost usando `curl localhost:9003/metrics` . 
```out
# HELP nginx_connections_accepted Accepted client connections
# TYPE nginx_connections_accepted counter
nginx_connections_accepted 33
# HELP nginx_connections_active Active client connections
# TYPE nginx_connections_active gauge
nginx_connections_active 1
# HELP nginx_connections_handled Handled client connections
# TYPE nginx_connections_handled counter
nginx_connections_handled 33
# HELP nginx_connections_reading Connections where NGINX is reading the request header
# TYPE nginx_connections_reading gauge
nginx_connections_reading 0
# HELP nginx_connections_waiting Idle client connections
# TYPE nginx_connections_waiting gauge
nginx_connections_waiting 0
# HELP nginx_connections_writing Connections where NGINX is writing the response back to the client
# TYPE nginx_connections_writing gauge
nginx_connections_writing 1
# HELP nginx_http_requests_total Total http requests
# TYPE nginx_http_requests_total counter
nginx_http_requests_total 94
# HELP nginx_up Status of the last metric scrape
# TYPE nginx_up gauge
nginx_up 1
# HELP nginxexporter_build_info Exporter build information
# TYPE nginxexporter_build_info gauge
nginxexporter_build_info{arch="linux/amd64",commit="e4a6810d4f0b776f7fde37fea1d84e4c7284b72a",date="2022-09-07T21:09:51Z",dirty="false",go="go1.19",version="0.11.0"} 1
```

#### Configurando o job no prometheus para coleta do nginx

Adicione no bloco scrape_configs do arquivo /etc/prometheus/prometheus.yml as configurações abaixo:

```yml
  - job_name: server_nginx
    scrape_interval: 5s
    static_configs:
      - targets: ['IP-Alvo:9003']
        labels:
          alias: nginx_server
```
#### Reinicie o prometheus 

```sh
sudo systemctl restart prometheus 
sudo systemctl status prometheus

``` 
#### Dashboard para visualizar as metricas 
Baixe o arquivo json  e import no grafana

```sh
curl -LO https://raw.githubusercontent.com/nginxinc/nginx-prometheus-exporter/main/grafana/dashboard.json
```
