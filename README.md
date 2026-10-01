# 1.0 Objetivo.

El enunciado de esta breve práctica consiste en:


# 2.0 Antes de empezar.

Necesitamos tener la práctica anterior hecha:

<a>https://github.com/iliangithub/IAM-Keycloak-oauth2-lab</a>

Que además, nos ayudarán los comandos anteriores para acceder a la máquina hecha en WSL.

Vamos a ver las máquinas que hay:

```
wsl -l -v
```

A continuación seleccionaremos la máquina en cuestión, en mi caso la Debian:

```
wsl -d Debian
```

Vamos a ver que tenemos el KeyCloak encendido:

```
docker ps
```

Si no lo está lo encendemos, necesitamos su ID:

```
docker ps -a
```

Encendemos:

```
docker start <id contenedor keycloak>
```

Recordemos además que vamos a usar Postman, por lo que lo necesitamos tener instalado.

# 3.0 Procedimientos

## 3.1 Preparar la WSL:

El entorno virtual no es burocracia: aísla las dependencias del proyecto del Python del sistema, y es lo que te permite tener un requirements.txt que otro reproduzca. En Debian y Ubuntu recientes, además, instalar paquetes de Python en el sistema está bloqueado a propósito, así que el entorno virtual es directamente obligatorio.

```
mkdir -p ~/iam-api && cd ~/iam-api
sudo apt install -y python3-venv
python3 -m venv .venv
source .venv/bin/activate
pip install fastapi uvicorn "pyjwt[crypto]" requests
```

## 3.2 Preparar KeyCloak.

Realm lab-iam, comprueba el selector de arriba a la izquierda antes de nada.

<h3>a) Crear el ámbito de cliente</h3>

Client scopes, Create client scope.

Name: iam-api
Description: Audiencia de la API de recursos
Type: Default
Protocol: OpenID Connect
Display on consent screen: Off
Include in token scope: On

Save.

b) Crear el mapeador dentro del ámbito

Ya dentro de iam-api, pestaña Mappers, botón Configure a new mapper, tipo Audience.

Name: audience-iam-api
Included Client Audience: vacío
Included Custom Audience: iam-api
Add to ID token: Off
Add to access token: On
Add to token introspection: On

Save.

Los dos campos de audiencia se excluyen. El primero es un desplegable con los clientes del realm y el segundo es texto libre, que es el que necesitas, porque tu API no está registrada como cliente: no pide tokens, solo los recibe.

c) Asignar el ámbito al cliente

Clients, spa-web, pestaña Client scopes, botón Add client scope, marcas iam-api y lo añades como Default.

Este paso es imprescindible y no es el mismo que el Type Default del punto a). Aquel solo dice qué ámbitos reciben los clientes nuevos, y spa-web ya existía.

d) Crear el rol

Realm roles, Create role, nombre auditor.

e) Asignar el rol al usuario

Users, cesar23, pestaña Role mapping, Assign role. Cambia el filtro a Realm roles para que aparezca auditor, márcalo y asigna.
