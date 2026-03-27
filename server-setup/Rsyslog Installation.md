# Rsyslog Installation

Es necesario el uso de rsyslog en el server Debian para recibir, formatear, y guardar en un archivo .log customizado los logs generados en pfSense.
 
 rsyslog ademas me permitio filtrar los logs para poder separar los generados por el firewall de pfsense y por el IDS/IPS Suricata dado que estan en formatos diferentes y ambos llegan por syslog en formato string.

Con respecto de los logs de Suricata en formato json, entiendo que es una buena practica con Wazuh usar un Wazuh Agent para enviarlos directamente con filebeat, sin embargo dado que Netgate limita bastante la instalacion de software de terceros en la configuracion de pfsense y cada vez que se reinicia el firewall tiende a restaurar la configuracion que esta indicada por los scripts de php-fpm y rompe la customizada, es por ello que segui lo recomendado por Netgate y envie los logs en formato json por syslog y luego los separe en el server que los recibe. Así los mapeo a Wazuh y comprobe que internamente Wazuh los decodifica bien y los muestra en el dashboard.

`###############`
`###TEMPLATE####`
`###############`

`template(name="suricata_json" type="string"`
`#    string="{\"date\":\"%timereported%\",\"hostname\":\"%hostname:R,ERE,1,DFLT:([^.]+)--end%\",\"program\":\"%programname:R,ERE,1,DFLT:([^.]+)--end%\",\"message\":%msg%}\n")`
     `string="%msg:2:$%\n")`

`# Formato: $DATE $HOST $PROGRAM: $MSG`
`template(name="pfsense_format" type="string"`
    `string="%timereported% %hostname:R,ERE,1,DFLT:([^.]+)--end% %programname%:%msg%\n")`

`###############`
`#ROUTING RULES#`
`###############`

`# Guarda los logs de suricata en formato json en /var/log/suricata-eve.log`
`if $inputname == "imudp" and $programname == "suricata" then {`
    `action(`
        `type="omfile"`
        `file="/var/log/suricata-eve.log"`
        `template="suricata_json"`
    `)`
    `stop`
`}`

`# Guardar todos los datos recibidos por UDP en /var/log/pfsense.log
`# en formato string y usando la plantilla definida arriba
if $inputname == "imudp" then {
    action(
        type="omfile"
        file="/var/log/pfsense.log"
        template="pfsense_format"
    )
    stop
}

## Logrotate

Ademas para que los logs no fueran un problema para mi almacenamiento use Logrotate para limitar su tamaño, comprimirlos y crear uno nuevo.

`/var/log/pfsense.log {`
    `size 50M       # rota cuando llega a 50MB`
    `rotate 2       # guarda solo 2 archivos historicos`
    `compress`
    `missingok`
    `notifempty`
    `postrotate`
        `systemctl kill -s HUP rsyslog`
    `endscript`
`}`

`/var/log/suricata-eve-log {`
    `size 100M`
    `rotate 2`
    `compress`
    `missingok`
    `notifempty`
    `postrotate`
        `systemctl kill -s HUP rsyslog`
    `endscript`
`}`