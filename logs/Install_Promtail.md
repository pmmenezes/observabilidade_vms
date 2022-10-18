# Instalação do Promtail 

### Instalação do pacotes e configuração do Promtail

```sh
 sudo apt install unzip
 curl -LO "https://github.com/grafana/loki/releases/download/v2.6.1/promtail-linux-amd64.zip" --output-dir /tmp
 
 unzip /tmp/promtail-linux-amd64.zip -d /tmp
 sudo mv /tmp/promtail-linux-amd64 /usr/local/bin
 sudo  chmod a+x /usr/local/bin/promtail-linux-amd64
 wget https://raw.githubusercontent.com/grafana/loki/master/clients/cmd/promtail/promtail-local-config.yaml -P /tmp
 sudo mv /tmp/promtail-local-config.yaml /usr/local/bin/

```
### Executando promtail como serviço 

```sh

sudo useradd -m promtail
sudo usermod -a -G adm promtail
sudo chown promtail:promtail /usr/local/bin/promtail-linux-amd64

```
### Arquivo de inicialização do promtail (promtail.service)
 
```
sudo vim /etc/systemd/system/promtail.service
```

```ini 

[Unit]
Description=promtail service
After=network.target

[Service]
Type=simple
User=promtail
ExecStart=/usr/local/bin/promtail-linux-amd64 -config.file /usr/local/bin/promtail-local-config.yaml

[Install]
WantedBy=multi-user.target

``` 
#### Ative o serviço promtail
```sh
sudo systemctl start promtail
sudo systemctl enable promtail
sudo systemctl status promtail

```
#### Teste o acesso 

`curl localhost:9080/metrics`