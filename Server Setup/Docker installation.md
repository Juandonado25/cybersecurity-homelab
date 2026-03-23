# Docker Installation

Una vez instalado el OS con la configuración básica y actualizado hay que instalar algunos paquetes básicos de utilidades del sistema que son necesarios para que Docker funcione correctamente.

La instalación se realizó usando los repositorios de apt. 

Primero se configuran los repositorios de docker:

````bash
# Add Docker's official GPG key:
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/debian
Suites: $(. /etc/os-release && echo "$VERSION_CODENAME")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
````

luego ejecutamos el comando `sudo apt install ca-certificates curl gnupg lsb-release -y` para instalar las utilidades necesarias en Debian.

Y luego para instalar docker con sus componentes se ejecuta el comando  `sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y` .

Para confirmar el estado de docker en el sistema usamos el comando `sudo systemctl status docker` y si no esta iniciado lo iniciamos con `sudo systemctl start docker`.

---
## Sources:

- [Debian \| Docker Docs](https://docs.docker.com/engine/install/debian/)

