Crie o arquivo `/etc/consul.d/webser.hcl` como contéudo abaixo

```ini
service {
  name = "webserver"
  id   = "nginx"
  port = 80
  tags = ["webserver"]

  meta = {
    enviroment = "nginx-lab01"
}
  check = {
            http = "http://172.31.12.7"
            interval = "10s"
        }

}
```
```sh 
consul reload 
```