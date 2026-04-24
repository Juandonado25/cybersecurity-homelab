# Telegram Notifications

En infraestructuras de seguridad críticas con disponibilidad 24/7, la implementación de sistemas de alerta temprana es fundamental para optimizar el MTTR (Mean Time To Repair). Para garantizar la relevancia de las notificaciones y reducir el ruido operacional, se filtraron únicamente las alertas de nivel 8 o superior.

Al tratarse de un entorno de laboratorio, opté por un enfoque pragmático utilizando Telegram como canal de mensajería por su simplicidad y portabilidad, permitiendo el monitoreo en tiempo real desde dispositivos móviles. Dado que la instancia de **Wazuh** se ejecuta en **Docker**, la configuración requirió el mapeo de volúmenes entre el host y el contenedor para asegurar la persistencia de los cambios. Como paso previo, es indispensable la creación de un bot mediante @BotFather y la obtención del API Token y el Chat ID correspondiente.

Como guía use la documentación y el script proporcionados por [Clockwork Computer](https://clockworkcomputerip.blogspot.com/2025/12/wazuh-telegram.html).

---
### Instalación:

Lo primero fue clonar el repositorio de que contiene los scripts en /tmp y pasarlos al directorio donde están guardados los archivos de configuración de Wazuh manager. esto lo hice desde el endpoint Debian pensado para la administración del server. si bien no esta en la imagen es importante darle los permisos necesarios al script para que wazuh lo pueda ejecutar correctamente.

![](../images/Wazuh-tg-1.png)

![](../images/wazuh-tg-2.png)

![](../images/wazuh-tg-3.png)

![](../images/wazuh-tg-4.png)

![](../images/wazuh-tg-5.png)

![](../images/wazuh-tg-6.png)

Y efectivamente llego el mensaje

![](../images/wazuh-tg-7.png)

---

### Prueba Real

Para esta prueba voy a generar un alerta de nivel 8 creando un usuario nuevo en el server que hace de host para el contenedor de wazuh, el cual tiene un wazuh-agent para monitorearlo.

![](../images/wazuh-tg-9.png)

![](../images/wazuh-tg-8.png)

![](../images/wazuh-tg-10.png)
 
![](../images/wazuh-tg-11.png)

**Wazuh envió el mensaje a Telegram correctamente**

Como dato adicional se puede ver como Suricata detecto y alerto que se estaba conectando a Telegram lo cual puede ser potencialmente perjudicial pero yo se que es trafico legitimo es por eso que decidí por el momento no tomar acción a pesar el ruido que puede generar.

![](../images/wazuh-tg-12.png)