# Instalação do Prometheus Server 

### Atualizar o  repositório de pacotes e atualizar o sistema

```bash
sudo apt update && sudo apt upgrade -y
```
### Baixar o Binário
A versão mais recente pode ser encontrada em https://prometheus.io/download/#prometheus
Durante a escrita deste Howto a versão mais atual é a [**2.39.1**](https://github.com/prometheus/prometheus/releases/download/v2.39.1/prometheus-2.39.1.linux-amd64.tar.gz)  mas para uso em produção e manter o documento mais tempo de suporte será utilizada a versão [**LTS 2.37.1**](https://github.com/prometheus/prometheus/releases/download/v2.37.1/prometheus-2.37.1.linux-amd64.tar.gz)

```bash
curl -LO https://github.com/prometheus/prometheus/releases/download/v2.37.1/prometheus-2.37.1.linux-amd64.tar.gz --output-dir /tmp

```
Ref.
[curl](https://curl.se/docs/manpage.html) 

### Criar usuário de grupo de serviço para gerenciar o prometheus

```bash
sudo addgroup --system prometheus
sudo adduser --shell /sbin/nologin --system --group prometheus
```

Ref. [Comandos para manipulação de contas](https://www.guiafoca.org/guiaonline/inicianteintermediario/ch12.html)

### Criar estruturas de diretórios e mover os arquivos e binários do prometheus
#### Diretorio de configuração 
```
sudo mkdir -p /etc/prometheus
``` 
#### Diretório de dados
```
sudo mkdir -p /var/lib/prometheus
``` 
#### Diretório de logs
```
sudo mkdir -p /var/log/prometheus
```
#### Extrair os arquivos e binários do prometheus e mover para os direótios definitivos
```
tar -zxvf /tmp/prometheus-*.tar.gz -C /tmp
```
#### Mover o binários prometheus e promtool  do para  /usr/local/bin/
```bash
sudo mv /tmp/prometheus-2.37.1.linux-amd64/{prometheus,promtool} /usr/local/bin/
sudo mv /tmp/prometheus-2.37.1.linux-amd64/{consoles,console_libraries} /etc/prometheus/
```

#### Criar configuração inicial do prometheus
Ref.  [Configuração do prometheus](https://prometheus.io/docs/prometheus/latest/configuration/configuration/)

Criar arquivo inicial de configuração  /etc/prometheus/prometheus.yml:

```bash

sudo bash -c ' cat <<FIM > /etc/prometheus/prometheus.yml
global:
  scrape_interval: 15s 
  evaluation_interval: 15s  
  scrape_timeout : 10s 

scrape_configs:
 
  - job_name: "prometheus"
    scrape_interval: 5s
    static_configs:
      - targets: ["localhost:9090"] 
FIM'
```
#### Abaixo segue um conteudo do aquirvo prometheus.yml com os mesmos valores **ativos** com comentários para entender cada um dos parâmtetros. 

```yml
# Configurações contidas no bloco *global* serão utilizadas em todos os jobs a nã ser que haja uma configuração específica no bloco do job.
global:
  scrape_interval: 15s # Definie o intervalo de coleta de dados. O padrão é a cada 1 minuto.
  evaluation_interval: 15s # Intervalo de tempos usado para verificar as regras de alertas. O padrão é a cada 1 minuto.
  scrape_timeout : 10s # Intervalo de tempo usado para condiderar que o alvo está indisponível.

# Configrações do Gerenciador de Alertas
#alerting:
#  alertmanagers:
#    - static_configs:
#        - targets:
          # - alertmanager:9093

#  
## Bloco rules_files é usado para definições de regras de alertas, as regras são carregadas uma vez e em intervalos peródivcos conforme o parâmetro evaluation_interval definido no bloco global 
#rule_files: 
  # - "first_rules.yml"
  # - "second_rules.yml"

# Bloco scrape_configs as configurações dos endpoints de coleta e a maneira que irá coletá-las.
scrape_configs:

    - job_name: "prometheus" # Nome do serviço dado ao prometheus para monitorar usado como rótuo (label) para coleta para qualquer série temporal extraída desta configuração
    scrape_interval: 5s # Definie o intervalo de coleta de dados específico para este job

    # O parâmetro metrics_path define o caminho para acesso as metricas o padrão é '/metrics'
    #metrics_path : "/metrics"
    # O parâmetro scheme define o protocolo usado para as requisições o padrão é http
    #scheme defaults: "http".

    static_configs:
      - targets: ["localhost:9090"] # Endpoint do alvo para coleta, neste caso o próprio prometheus


```

#### Mudar a propriedade dos aquivos e dieretórios do prometheus para o user prometheus

```
sudo chown -R prometheus:prometheus /var/log/prometheus /etc/prometheus \
                                    /var/lib/prometheus /usr/local/bin/{prometheus,promtool}

```
#### Criando um serviço do systemd para o prometheus

```bash
sudo vim /etc/systemd/system/prometheus.service
```

```ini
[Unit]
Description=Prometheus
Documentation=https://prometheus.io/docs/introduction/overview/
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecReload=/bin/kill -HUP $MAINPID
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090 \
 
SyslogIdentifier=prometheus
Restart=always

[Install]
WantedBy=multi-user.target  

```

```ini

[Unit] # Bloco de definição do serviço.
Description=Prometheus # Nome do serviço
Documentation=https://prometheus.io/docs/introduction/overview/ # Documentação do serviço.
Wants=network-online.target # Requisito para ser iniciado, no caso o serviço de rede precisa estar ativo.
After=network-online.target # Serviço será inciado somente após o serviço de rede.

[Service] # Bloco de definição de inicialização do serviço.
Type=simple # Tipo do serviço, o padrão é simple. Pode conter  subserviços.
User=prometheus # Usuário do serviço
Group=prometheus # Grupo do serviço
ExecReload=/bin/kill -HUP $MAINPID # O serviço do Prometheus será reiniciado ao receber um sinal de reinicialização.
ExecStart=/usr/local/bin/prometheus \ # Comando para o serviço  ser iniciado
  --config.file=/etc/prometheus/prometheus.yml \ 
  --storage.tsdb.path=/var/lib/prometheus \ 
  --web.console.templates=/etc/prometheus/consoles \ 
  --web.console.libraries=/etc/prometheus/console_libraries \ 
  --web.listen-address=0.0.0.0:9090 \ 
  
```

```sh

sudo systemctl start prometheus

sudo systemctl enable prometheus

sudo systemctl status prometheus

```
Caso possua firewall ativado liberar o trafego ` sudo ufw allow 9090/tcp`

Acessar o Servidor ` http://<ip address>or<hostname>:9090 `








