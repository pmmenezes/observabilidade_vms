```sh
curl -L https://github.com/consul-up/birdwatcher/releases/download/v1.0.0/backend-linux-amd64 --output-dir /tmp
curl -L https://github.com/consul-up/birdwatcher/releases/download/v1.0.0/frontend-linux-amd64 --output-dir /tmp
```

```sh
sudo mv /tmp/backend-linux-amd64 /usr/local/bin/backend
sudo mv /tmp/frontend-linux-amd64  /usr/local/bin/frontend
sudo chmod +x /usr/local/bin/{frontend,backend}
```

```sh
sudo vim  /etc/systemd/system/frontend.service
```
```ini
[Unit]
Description="Frontend service"
# The service requires the VM's network
# to be configured, e.g., an IP address has been assigned.
Requires=network-online.target
After=network-online.target
[Service]
# ExecStart is the command to run.
ExecStart=/usr/local/bin/frontend
# Restart configures the restart policy. In this case, we
# want to restart the service if it fails.
Restart=on-failure
# Environment sets environment variables.
# We will set the frontend service to listen
# on port 6060.
Environment=BIND_ADDR=0.0.0.0:6060
# We set BACKEND_URL to http://localhost:7000 because
# that's the port we'll run our backend service on.
Environment=BACKEND_URL=http://localhost:7000
# The Install section configures this service to start
# automatically if the VM reboots.
[Install]
WantedBy=multi-user.target
```

```sh
sudo vim /etc/systemd/system/backend.service
```

```ini
[Unit]
Description="Backend service"
Requires=network-online.target
After=network-online.target
[Service]
ExecStart=/usr/local/bin/backend
Restart=on-failure
# We will set the backend service to listen
# on port 7000.
Environment=BIND_ADDR=0.0.0.0:7000
[Install]
WantedBy=multi-user.target
```

```sh
sudo systemctl start frontend
sudo systemctl enable frontend  
sudo systemctl status frontend 
sudo systemctl start  backend
sudo systemctl enable  backend
sudo systemctl status  backend

``` 
Teste acessando `http://IP-PUBLICO:6060/`

## Registrando o serviço no consul

```sh
sudo vim /etc/consul.d/frontend.hcl
``` 

```ini
service {
    name = "frontend"
# frontend runs on port 6060.
    port = 6060
# The "connect" stanza configures service mesh
# features.
    connect {
        sidecar_service {
        # frontend's proxy will listen on port 21000.
            port = 19000
            proxy {
            # The "upstreams" stanza configures
            # which ports the sidecar proxy will expose
            # and what services they'll route to.
            upstreams = [
                {
                # Here you're configuring the sidecar proxy to
                # proxy port 6001 to the backend service.
                destination_name = "backend"
                local_bind_port = 6001
                }
            ]
        }
    }
}
}
``` 

```sh 
sudo vim /etc/consul.d/backend.hcl
```
```ini
service {
    name = "backend"
    # backend runs on port 7000.
    port = 7000
    meta {
        version = "v1"
    }
    # The backend service doesn't call
    # any other services so it doesn't
    # need an "upstreams" stanza.
    #
    # The connect stanza is still required to
    # indicate that it needs a sidecar proxy.
    connect {
        sidecar_service {
        # backend's proxy will listen on port 22000.
        port = 19001
        }
    }
}
```
```sh
consul reload
``` 

```
sudo vim /etc/systemd/system/frontend-sidecar-proxy.service
```
```ini
[Unit]
Description="Frontend sidecar proxy service"
Requires=network-online.target
After=network-online.target
[Service]
ExecStart=/usr/bin/consul connect envoy -sidecar-for frontend \
-admin-bind 127.0.0.1:19000
Restart=on-failure
[Install]
WantedBy=multi-user.target
And /etc/systemd/system/backend-sidecar-proxy.service should match Example 4-10.
Example 4-10. /etc/systemd/system/backend-sidecar-proxy.service
[Unit]
Description="Backend sidecar proxy service"
Requires=network-online.target
After=network-online.target
[Service]
ExecStart=/usr/bin/consul connect envoy -sidecar-for backend \
-admin-bind 127.0.0.1:19001
Restart=on-failure
[Install]
WantedBy=multi-user.target
```

```sh
sudo systemctl start frontend-sidecar-proxy
sudo systemctl enable frontend-sidecar-proxy
sudo systemctl status frontend-sidecar-proxy
```

```sh
sudo vim /etc/systemd/system/backend-sidecar-proxy.service
```
```ini
[Unit]
Description="Backend sidecar proxy service"
Requires=network-online.target
After=network-online.target
[Service]
ExecStart=/usr/bin/consul connect envoy -sidecar-for backend \
-admin-bind 127.0.0.1:19001
Restart=on-failure
[Install]
WantedBy=multi-user.target
``` 
```sh
sudo systemctl start backend-sidecar-proxy
sudo systemctl enable backend-sidecar-proxy
sudo systemctl status backend-sidecar-proxy
```
Instale o [envoy](../../envoy_install.md) e altere a porta da variável `Environment=BIND_ADDR=0.0.0.0:6060` para `Environment=BIND_ADDR=0.0.0.0:6001` em  `/etc/systemd/system/frontend.service`


