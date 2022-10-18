# Observabilidade de seu ambiente com COnsul Hashicorp
## Consul  para ambiente de produção
### Instalação do Consul

```sh
sudo apt update && sudo apt upgrade -y
```

```sh
sudo apt install curl gnupg

curl --fail --silent --show-error --location https://apt.releases.hashicorp.com/gpg | \
      gpg --dearmor | \
      sudo dd of=/usr/share/keyrings/hashicorp-archive-keyring.gpg
      
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
 sudo tee -a /etc/apt/sources.list.d/hashicorp.list
 
 sudo apt-get update
```
```sh
sudo apt-cache policy consul
sudo apt-get install consul=1.13.2-1
consul
``` 
### Preparando as credencias de segurança para comunicação entres agentes ( chaves de criptografia e certificados)

```sh
consul keygen
WOXlWw/FkjciZxVvgia4jMF6uELPBTJSc0plRlDGnYk=
```

```sh
consul tls ca create # -domain local
```

```sh
consul tls cert create -server -dc lab01 # -domain local
```

```sh
sudo cp  consul-agent-ca* lab01-server-consul-0* /etc/consul.d/
```

#```sh
#consul tls cert create -client -dc lab01 # -domain local
#```
#### Distribua as chaves e certificados para os servidores consul e para os clientes
 
```sh
scp consul-agent-ca.pem lab01-server-coonsul-0.pem lab01-server-consul-0-key.pem ubuntu@<IP-SERVER>:/etc/consul.d/
```
```sh
scp consul-agent-ca.pem ubuntu@<IP-CLIENTES>:/etc/consul.d/
```

#
#```sh
#scp consul-agent-ca.pem lab01-client-consul-0.pem lab01-client-coonsul-0-key.pem #ubuntu@<IP-CLIENTS>:/etc/consul.d/
#```

### Cofigurando os Agentes
 
 ```sh
sudo mv /etc/consul.d/consul.hcl /etc/consul.d/consul.hcl.origin
sudo vim  /etc/consul.d/consul.hcl
```
```ini
datacenter = "lab01"
data_dir = "/opt/consul"
encrypt = "WOXlWw/FkjciZxVvgia4jMF6uELPBTJSc0plRlDGnYk="
verify_incoming = true
verify_outgoing = true
verify_server_hostname = true

ca_file = "/etc/consul.d/consul-agent-ca.pem"
cert_file = "/etc/consul.d/lab01-server-consul-0.pem"
key_file = "/etc/consul.d/lab01-server-consul-0-key.pem"

auto_encrypt {
  allow_tls = true
}

#acl {
#  enabled = true
#  default_policy = "allow"
#  enable_token_persistence = true
#}
#
#performance {
#  raft_multiplier = 1
#}
```

```sh
sudo chown --recursive consul:consul /etc/consul.d
sudo chmod 640 /etc/consul.d/consul.hcl
```



```sh

sudo vim /etc/consul.d/server.hcl 

```
Edite o arquivo `/etc/consul.d/server.hcl` nos **Server**

```ini
server = true
bootstrap_expect = 1
bind_addr = "IP_PRIVATE"
retry_join =["IPs ou Hostname dos Servers"]
client_addr = "0.0.0.0"
enable_script_checks = true

connect {
  enabled = true
}

#addresses {
#  grpc = "127.0.0.1"
#}

ports {
  grpc  = 8502
}

ui_config {
  enabled = true
}
```
```sh
sudo chown --recursive consul:consul /etc/consul.d
sudo chmod 640 /etc/consul.d/server.hcl
``` 

```sh
sudo vim /etc/systemd/system/consul.service

```
```ini
[Unit]
Description="HashiCorp Consul - A service mesh solution"
Documentation=https://www.consul.io/
Requires=network-online.target
After=network-online.target
ConditionFileNotEmpty=/etc/consul.d/consul.hcl

[Service]
EnvironmentFile=-/etc/consul.d/consul.env
User=consul
Group=consul
ExecStart=/usr/bin/consul agent -config-dir=/etc/consul.d/
ExecReload=/bin/kill --signal HUP $MAINPID
KillMode=process
KillSignal=SIGTERM
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```
```sh
sudo systemctl start consul
sudo systemctl enable consul
sudo systemctl status consul
```

#

##```sh
##export CONSUL_CACERT=/etc/consul.d/consul-agent-ca.pem
#export CONSUL_CLIENT_CERT=/etc/consul.d/<dc-name>-<server/#client>-consul-<cert-number>.pem
#export CONSUL_CLIENT_KEY=/etc/consul.d/<dc-name>-<server/#client>-consul-<cert-number>-key.pem
#```
`sudo rm /opt/consul/serf/local.keyring`


#### Referências
[Day 1: Deploy Your First Datacenter](https://learn.hashicorp.com/collections/consul/production-deploy)

[How to Use ssh-keygen to Generate a New SSH Key?](https://www.ssh.com/academy/ssh/keygen#what-is-ssh-keygen?)