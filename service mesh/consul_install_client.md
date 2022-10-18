## Instalação do Consul

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
```sh
scp consul-agent-ca.pem ubuntu@<IP-CLIENTES>:/tmp
sudo cp /tmp/consul-agent-ca.pem /etc/consul.d/
```

 ```sh
sudo mv /etc/consul.d/consul.hcl /etc/consul.d/consul.hcl.origin
sudo vim  /etc/consul.d/consul.hcl
```

Edite o arquivo `/etc/consul.d/consul.hcl/etc/consul.d/consul.hcl` nos **Clientes**

```ini
datacenter = "lab01"
server = false
retry_join = ["IP-SERVER"]
data_dir = "/opt/consul"
encrypt = "WOXlWw/FkjciZxVvgia4jMF6uELPBTJSc0plRlDGnYk="
verify_incoming = true
verify_outgoing = true
verify_server_hostname = true
enable_script_checks = true

ca_file = "/etc/consul.d/consul-agent-ca.pem"

auto_encrypt {
  tls = true
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
sudo chown --recursive consul:consul /etc/consul.d
```

```sh
sudo systemctl start consul
sudo systemctl enable consul
sudo systemctl status consul
```