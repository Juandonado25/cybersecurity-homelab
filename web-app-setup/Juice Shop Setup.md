OWASP Juice Shop es una aplicación web intencionadamente insegura desarrollada por el Open Web Application Security Project (OWASP). Está diseñada como una plataforma moderna y realista para practicar pruebas de penetración de aplicaciones web en un entorno seguro y legal.

Para la instalacion en docker sigo los pasos indicados en la documentacion:

	docker pull bkimminich/juice-shop
	run -d --rm -p 3000:3000 bkimminich/juice-shop

Luego en el navegador, simulando acceso desde una red WAN desde fuera de la red de Virtualbox, accedo a la IP de la VM de pfsense que esta puenteada. En mi caso no tuve que especificar el puerto porque hago port forwarding de los puertos 80 y 443 al puerto 3000 que es donde esta la aplicación web.

<img width="729" height="485" alt="image" src="/images/juice-shop-front.png" />

---

## Sources

- https://owasp.org/www-project-juice-shop/
- https://hub.docker.com/r/bkimminich/juice-shop#docker-container
