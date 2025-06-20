# Step-by-step

First, check the prerequisites document to ensure your environment is properly configured.

# Manual

1. Access your Zabbix dashboard, create a Host, and add the template provided in this repository.

2. With access to your Zabbix server (Ubuntu/Debian), install the following packages:

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

2. Then create the script that will run the speed test and send the data to your Zabbix server

- You will need to change the following variables:

IP_ZABBIX = Enter the IP of your Zabbix server. If the script is executed on the same machine as your server, use 127.0.0.1 

HOST_ZABBIX = nter the name of the Host you created in Zabbix that will receive the script data. *Note:* keep the double quotes "".

```
# Run the test and capture the JSON
RESULT=$(speedtest --json)

# Extract and convert values with jq
DOWNLOAD=$(echo "$RESULT" | jq '.download / 1000000')
UPLOAD=$(echo "$RESULT" | jq '.upload / 1000000')
PING=$(echo "$RESULT" | jq '.ping')
LATENCY=$(echo "$RESULT" | jq '.server.latency')
SERVER_HOST=$(echo "$RESULT" | jq -r '.server.host')
SERVER_CITY=$(echo "$RESULT" | jq -r '.server.name')
ISP=$(echo "$RESULT" | jq -r '.client.isp')
IP=$(echo "$RESULT" | jq -r '.client.ip')

# Send data to Zabbix
zabbix_sender -z IP_ZABBIX -s "HOST_ZABBIX" -k speedtest.download -o "$DOWNLOAD"
zabbix_sender -z IP_ZABBIX -s "HOST_ZABBIX" -k speedtest.upload -o "$UPLOAD"
zabbix_sender -z IP_ZABBIX -s "HOST_ZABBIX" -k speedtest.ping -o "$PING"
zabbix_sender -z IP_ZABBIX -s "HOST_ZABBIX" -k speedtest.latency -o "$LATENCY"
zabbix_sender -z IP_ZABBIX -s "HOST_ZABBIX" -k speedtest.server_host -o "$SERVER_HOST"
zabbix_sender -z IP_ZABBIX -s "HOST_ZABBIX" -k speedtest.server_city -o "$SERVER_CITY"
zabbix_sender -z IP_ZABBIX -s "HOST_ZABBIX" -k speedtest.isp -o "$ISP"
zabbix_sender -z IP_ZABBIX -s "HOST_ZABBIX" -k speedtest.client_ip -o "$IP"
```

3. Then save the file with the .sh extension

4. Make the file executable

``
chmod +x "Filename"
``

5. Schedule the script to run in crontab.

``
*/5 * * * * "script path"
``

6. Access your Zabbix dashboard and verify data reception.

# Automático

1. Access your Zabbix server, create a Host, and add the template provided in this repository.

2. On your server's terminal, create the following script with a .sh extension. 

```
#!/bin/bash

echo "Installing dependencies..."

sudo apt-get update -y

sudo apt-get install -y jq

sudo apt-get install -y zabbix-sender

sudo apt-get install -y curl

sudo curl -s https://packagecloud.io/install/repositories/ookla/speedtest-cli/script.deb.sh | sudo bash

sudo apt-get install -y speedtest-cli

echo "Dependencies successfully installed"

echo "***************************Script generator***************************************"

read -p "Enter the Host name as registered in Zabbix:: " HOST_ZABBIX

read -p "Enter the name of your script file: " NOME_ARQUIVO

read -p "Enter the IP address of your Zabbix server: " IP_ZABBIX

ARQUIVO="${NOME_ARQUIVO}.sh"

cat <<EOF > "$ARQUIVO"
#!/bin/bash

# Run the test and capture the JSON
RESULT=\$(speedtest --json)

# Extract and convert values with jq
DOWNLOAD=\$(echo "\$RESULT" | jq '.download / 1000000')
UPLOAD=\$(echo "\$RESULT" | jq '.upload / 1000000')
PING=\$(echo "\$RESULT" | jq '.ping')
LATENCY=\$(echo "\$RESULT" | jq '.server.latency')
SERVER_HOST=\$(echo "\$RESULT" | jq -r '.server.host')
SERVER_CITY=\$(echo "\$RESULT" | jq -r '.server.name')
ISP=\$(echo "\$RESULT" | jq -r '.client.isp')
IP=\$(echo "\$RESULT" | jq -r '.client.ip')

# Send data to Zabbix
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
echo "Script created successfully: $ARQUIVO"

CAMINHO_ABSOLUTO="$(realpath "$ARQUIVO")"


if crontab -l 2>/dev/null | grep -q "$CAMINHO_ABSOLUTO"; then
    echo "An entry already exists in crontab for this script."
else
    (crontab -l 2>/dev/null; echo "*/5 * * * * $CAMINHO_ABSOLUTO") | crontab -
    echo "Script '$ARQUIVO' added to crontab to run every 5 minutes."
fi

echo "Script gerado com sucesso: $ARQUIVO"
```

3. Access your Zabbix dashboard and verify data reception.
