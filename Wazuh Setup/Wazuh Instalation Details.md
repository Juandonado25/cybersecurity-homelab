## Wazuh Instalation

Después de que este completa la instalación de Docker hay que instalar Wazuh, para esto la documentacion de Wazuh nos ofrece la opcion de clonar el repositorio usando `git clone https://github.com/wazuh/wazuh-docker.git -b v4.14.3`

Aumentamos areas de memoria como lo pide la documentacion con `sudo sysctl -w vm.max_map_count=262144` y para hacerlo persistente `echo "vm.max_map_count=262144" | sudo tee /etc/sysctl.d/99-wazuh.conf`

Vamos al directorio "single-node" usando `cd wazuh-docker/single-node/`

Generamos los certificados necesarios con el comando `docker compose -f generate-indexer-certs.yml run --rm generator`

Y por ultimo levantamos todos los componentes usando `docker compose up -d`

## Wazuh Setup 

Para que Wazuh funcione correctamente en mi laboratorio tuve que hacer las siguientes modificaciones:

La clave de la API fue necesario cambiarla porque la anterior generaba problemas por un '\*\' que se mal interpretaba.

se mapeo el filebeat para reemplazarlo dentro del contenedor de wazuh manager.

Cambie el puerto de 443 a 8443 para que evitar problemas con el uso de otros servicios.
### docker-compose.yml:
`services:`
  `wazuh.manager:`
    `ports:`
      `-  "514:514/udp"`
    `environment:`
      `- API_PASSWORD=Wazuh-Secret-2026!`
    `volumes:`
      `- ./config/filebeat.yml:/etc/filebeat/filebeat.yml:ro`
  `wazuh.dashboard:`
    `ports:`
      `- 8443:5601`
    `environment:`
      `- API_PASSWORD=Wazuh-Secret-2026!`

### ./config/filebeat.yml:

Fue necesario cambiar el parámetro de archives a "true".

Se tuvo que agregar a mano los valores de los parametros de los certificados.

`# Wazuh - Filebeat configuration file`
`filebeat.modules:`
  - `module: wazuh`
    `alerts:`
      `enabled: true`
    `archives:`
      `enabled: true`

`setup.template.json.enabled: true`
`setup.template.overwrite: true`
`setup.template.json.path: '/etc/filebeat/wazuh-template.json'`
`setup.template.json.name: 'wazuh'`
`setup.ilm.enabled: false`
`output.elasticsearch:`
  `username: "admin"`
  `password: "SecretPassword"`
  `hosts: ['https://wazuh.indexer:9200']`
  `ssl.enabled: true`
  `ssl.verification_mode: certificate`
  `ssl.certificate_authorities: ["/etc/ssl/root-ca.pem"]`
  `ssl.certificate: "/etc/ssl/filebeat.pem"`
  `ssl.key: "/etc/ssl/filebeat.key"`

`logging.metrics.enabled: false`

`seccomp:`
  `default_action: allow`
  `syscalls:`
  - `action: allow`
    `names:`
    - `rseq`
### ./config/wazuh_cluster/wazuh_manager.conf

Lo primero en este archivo es unificar los bloques con las etiquetas \<ossec_config\> y \<global\> para que quede todo en uno solo respectivamente.

Y luego me asegure de cambiar estos parametros para que reciba correctamente los logs de pfsense.

  `<global>`
    `<logall>yes</logall>`
    `<logall_json>yes</logall_json>`
  `<\global>`

  `<remote>`
    `<connection>syslog</connection>`
    `<port>514</port>`
    `<protocol>udp</protocol>`
    `<allowed-ips>0.0.0.0/0</allowed-ips>`
  `</remote>`

  `<cluster>`
    `<name>wazuh</name>`
    `<node_name>node01</node_name>`
    `<node_type>master</node_type>`
    `<key>aa093264ef885029653eea20dfcf51ae</key>`
    `<port>1516</port>`
    `<bind_addr>0.0.0.0</bind_addr>`
    `<nodes>`
        `<node>wazuh.manager</node>`
    `</nodes>`
    `<hidden>no</hidden>`
    `<disabled>yes</disabled>`
  `</cluster>`

### ./config/wazuh_dashboard/wazuh.yml

Se cambio la clade de la API.

`hosts:`
  - `1513629884013:`
      `url: "https://wazuh.manager"`
      `port: 55000`
      `username: wazuh-wui`
      `password: "Wazuh-Secret-2026!"`
      `run_as: true`

### Puertos expuestos por componentes centrales

- **1514/TCP** — Wazuh (comunicación agentes)

- **1515/TCP** — Wazuh (enrollment/registro agentes)

- **514/UDP** — Wazuh (syslog)

- **55000/TCP** — **API del Wazuh server**

- **9200/TCP** — **API del Wazuh indexer**

- **8443/TCP** — **Wazuh dashboard (HTTPS)**

### Configuracion de pfsense

Para que pfsende envie los logs hay que ir a **Status -> System Logs -> Settings** bajar hasta la seccion **Remote Logging Options** y modificar lo siguiente:
- Send log messages to remote syslog server: ok
- Source Addres: DMZ
- IP Protocol IPv4
- Remote log services: 10.0.0.50:514
- Remote Syslog Contents: System Events, Firewall Events, y DNS Events.

Es necesario por ahora dejar que los envie en formato "BSD (RFC 3164, default)" porque el wazuh puede no poder parsear el otro formato mas moderno.

**Compretar**
- Explicar como agregar archoves al dashboard
- agregar el decoder tanto en la carpeta como en el docker compose 
- 

---
## Sources:

- [Deployment on Docker - Installation alternatives · Wazuh documentation](https://documentation.wazuh.com/current/deployment-options/docker/index.html)
- [CLOCKWORK COMPUTER - WAZUH - pfSense](https://clockworkcomputerip.blogspot.com/2025/11/wazuh-pfsense.html)
- [Clockwork Computer Notes](https://drive.google.com/file/d/1i-bb766uI6W9goYvDlxvmBGnRlKk1Qp8/view)
- 