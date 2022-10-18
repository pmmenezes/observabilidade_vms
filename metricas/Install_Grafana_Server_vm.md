# Instalando Grafana

### Instalando requisitos necessários 

```sh
sudo apt-get install -y apt-transport-https
sudo apt-get install -y software-properties-common wget
```

### Adicionando Repositório do grafana OSS

```sh
sudo wget -q -O /usr/share/keyrings/grafana.key https://packages.grafana.com/gpg.key
echo "deb [signed-by=/usr/share/keyrings/grafana.key] https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list

```

### Instalação do grafana via apt 
```sh
sudo apt-get update
sudo apt-get install grafana -y
```

### Ativando o serviço do grafana

```sh
sudo systemctl start grafana-server
sudo systemctl enable grafana-server.service
sudo systemctl status grafana-server

```

Acessar o Servidor ` http://<ip address>or<hostname>:3000 `