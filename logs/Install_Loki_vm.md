# Instalação do Loki

### Instalação do pacotes e configuração do Loki

```
 sudo apt install unzip
 curl -LO "https://github.com/grafana/loki/releases/download/v2.6.1/loki-linux-amd64.zip" --output-dir /tmp
 unzip /tmp/loki-linux-amd64.zip -d /tmp
 sudo mv /tmp/loki-linux-amd64 /usr/local/bin
 sudo  chmod a+x /usr/local/bin/loki-linux-amd64
 wget https://raw.githubusercontent.com/grafana/loki/v2.6.1/cmd/loki/loki-local-config.yaml -P /tmp
 sudo mv /tmp/loki-local-config.yaml /usr/local/bin/

```
### Executando Loki como serviço 

```sh

sudo useradd -m loki
sudo chown loki:loki /usr/local/bin/loki-linux-amd64

```
### Arquivo de inicialização do Loki (loki.service)

```sh
sudo vim  /etc/systemd/system/loki.service
```
```ini 

[Unit]
Description=Loki service
After=network.target

[Service]
Type=simple
User=loki
ExecStart=/usr/local/bin/loki-linux-amd64 -config.file /usr/local/bin/loki-local-config.yaml

[Install]
WantedBy=multi-user.target

``` 
#### Ative o serviço loki
```
sudo systemctl start loki
sudo systemctl enable loki
sudo systemctl status loki
```
#### Teste o acesso 

`curl localhost:3100/metrics`

