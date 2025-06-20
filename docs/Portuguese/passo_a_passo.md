# Passo a passo

Primeiramente, veja o documento de pré requisistos para que você tenha o ambiente configurado.

# Manual

1. Acesse o seu painel do zabbix, crie um Host e adicione o template disponibilizado no repositório.

![Host](imgs/Host_zabbix.png)

3. Com acesso ao seu servidor zabbix (Ubuntu/Debian), realize a instalação das seguintes dependências:

- Speedtest CLI

> https://www.speedtest.net/pt/apps/cli

```
>sudo apt-get update -y
>sudo apt-get install -y speedtest-cli
```

- Zabbix sender

```
>sudo apt-get update -y
> sudo apt-get install zabbix-sender
```

- Jq

```
>sudo apt-get update -y
>sudo apt-get install -y jq
```

2. Após isso crie o script que irá executar o teste de velocidade e enviar os dados para o seu servidor zabbix

- Será necessário alterar as seguintes variáveis:

IP_ZABBIX = Insira o Ip do seu servidor zabbix, caso o script seja executado no mesmo local do seu servidor, insira 127.0.0.1 

HOST_ZABBIX = Insira o nome do Host que você criou no zabbix que irá receber as informações do script. Obs: mantenha as aspas "".

```
# Executa o teste e captura o JSON
RESULT=$(speedtest --json)

# Extrai e converte valores com jq
DOWNLOAD=$(echo "$RESULT" | jq '.download / 1000000')
UPLOAD=$(echo "$RESULT" | jq '.upload / 1000000')
PING=$(echo "$RESULT" | jq '.ping')
LATENCY=$(echo "$RESULT" | jq '.server.latency')
SERVER_HOST=$(echo "$RESULT" | jq -r '.server.host')
SERVER_CITY=$(echo "$RESULT" | jq -r '.server.name')
ISP=$(echo "$RESULT" | jq -r '.client.isp')
IP=$(echo "$RESULT" | jq -r '.client.ip')

# Envia os dados para o Zabbix
zabbix_sender -z IP_ZABBIX -s "HOST_ZABBIX" -k speedtest.download -o "$DOWNLOAD"
zabbix_sender -z IP_ZABBIX -s "HOST_ZABBIX" -k speedtest.upload -o "$UPLOAD"
zabbix_sender -z IP_ZABBIX -s "HOST_ZABBIX" -k speedtest.ping -o "$PING"
zabbix_sender -z IP_ZABBIX -s "HOST_ZABBIX" -k speedtest.latency -o "$LATENCY"
zabbix_sender -z IP_ZABBIX -s "HOST_ZABBIX" -k speedtest.server_host -o "$SERVER_HOST"
zabbix_sender -z IP_ZABBIX -s "HOST_ZABBIX" -k speedtest.server_city -o "$SERVER_CITY"
zabbix_sender -z IP_ZABBIX -s "HOST_ZABBIX" -k speedtest.isp -o "$ISP"
zabbix_sender -z IP_ZABBIX -s "HOST_ZABBIX" -k speedtest.client_ip -o "$IP"
```

3. Após isso, salve o arquivo com a extensão ".sh"

4. Libere a execução do arquivo

``
chmod +x "Nome do arquivo"
``

5. Configure a execução do script no crontab

``
*/5 * * * * "caminho do script"
``

6. Acesse o seu painel zabbix e verifique o recebimento dos dados.

# Automático

1. Acesse o seu servidor zabbix, crie um Host e adicione o template disponibilizado no repositório.

2. No terminal do seu servidor, crie o seguinte script com a extensão ".sh". 

```
#!/bin/bash

echo "Iniciando a instalação das dependências..."

sudo apt-get update -y

sudo apt-get install -y jq

sudo apt-get install -y zabbix-sender

sudo apt-get install -y curl

sudo curl -s https://packagecloud.io/install/repositories/ookla/speedtest-cli/script.deb.sh | sudo bash

sudo apt-get install -y speedtest-cli

echo "Dependências instaladas com sucesso"

echo "***************************Script generator***************************************"

read -p "Digite o nome do Host como esta escrito no zabbix: " HOST_ZABBIX

read -p "Insira o nome do seu arquivo: " NOME_ARQUIVO

read -p "Insira o endereço de ip do seu servidor zabbix: " IP_ZABBIX

ARQUIVO="${NOME_ARQUIVO}.sh"

cat <<EOF > "$ARQUIVO"
#!/bin/bash

# Executa o teste e captura o JSON
RESULT=\$(speedtest --json)

# Extrai e converte valores com jq
DOWNLOAD=\$(echo "\$RESULT" | jq '.download / 1000000')
UPLOAD=\$(echo "\$RESULT" | jq '.upload / 1000000')
PING=\$(echo "\$RESULT" | jq '.ping')
LATENCY=\$(echo "\$RESULT" | jq '.server.latency')
SERVER_HOST=\$(echo "\$RESULT" | jq -r '.server.host')
SERVER_CITY=\$(echo "\$RESULT" | jq -r '.server.name')
ISP=\$(echo "\$RESULT" | jq -r '.client.isp')
IP=\$(echo "\$RESULT" | jq -r '.client.ip')

# Envia os dados para o Zabbix
zabbix_sender -z "$IP_ZABBIX" -s "$HOST_ZABBIX" -k speedtest.download -o "\$DOWNLOAD"
zabbix_sender -z "$IP_ZABBIX" -s "$HOST_ZABBIX" -k speedtest.upload -o "\$UPLOAD"
zabbix_sender -z "$IP_ZABBIX" -s "$HOST_ZABBIX" -k speedtest.ping -o "\$PING"
zabbix_sender -z "$IP_ZABBIX" -s "$HOST_ZABBIX" -k speedtest.latency -o "\$LATENCY"
zabbix_sender -z "$IP_ZABBIX" -s "$HOST_ZABBIX" -k speedtest.server_host -o "\$SERVER_HOST"
zabbix_sender -z "$IP_ZABBIX" -s "$HOST_ZABBIX" -k speedtest.server_city -o "\$SERVER_CITY"
zabbix_sender -z "$IP_ZABBIX" -s "$HOST_ZABBIX" -k speedtest.isp -o "\$ISP"
zabbix_sender -z "$IP_ZABBIX" -s "$HOST_ZABBIX" -k speedtest.client_ip -o "\$IP"
EOF

chmod +x "$ARQUIVO"
echo "Script gerado com sucesso: $ARQUIVO"

CAMINHO_ABSOLUTO="$(realpath "$ARQUIVO")"


if crontab -l 2>/dev/null | grep -q "$CAMINHO_ABSOLUTO"; then
    echo "Já existe uma entrada no crontab para esse script."
else
    (crontab -l 2>/dev/null; echo "*/5 * * * * $CAMINHO_ABSOLUTO") | crontab -
    echo "Script '$ARQUIVO' adicionado ao crontab para rodar a cada 5 minutos."
fi

echo "Script gerado com sucesso: $ARQUIVO"
```

3. Acesse o seu painel do zabbix e verifique o recebimento dos dados
