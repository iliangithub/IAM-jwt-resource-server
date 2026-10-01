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

Lo primero estar en el realm `lab-iam`.

<img width="320" height="126" alt="Captura de pantalla 2026-09-29 131212" src="https://github.com/user-attachments/assets/494b2448-f189-4b69-b6a1-83e94322d009" />

### a) Crear el ámbito de cliente

<img width="648" height="220" alt="Captura de pantalla 2026-09-29 131431" src="https://github.com/user-attachments/assets/603399cd-57ed-4729-9330-8c48c95359e3" />

Nos vamos a client scopes.

Client scopes, Create client scope.
- Name: `iam-api`
- Description: `Audiencia de la API de recursos`
- Type: `Default`
- Protocol: `OpenID Connect`
- Display on consent screen: `Off`
- Include in token scope: `On`
- Include in OpenID Provider Metadata: `On`

<img width="948" height="756" alt="Captura de pantalla 2026-09-29 131558" src="https://github.com/user-attachments/assets/7c682aee-c7aa-4b5d-b288-821269470ce5" />

### b) Crear el mapeador dentro del ámbito

Ya dentro de iam-api, pestaña Mappers, tipo Audience.

<img width="1372" height="657" alt="Captura de pantalla 2026-09-29 135110" src="https://github.com/user-attachments/assets/987856c2-91c8-4675-b96e-273181f58507" />

Le damos al botón `Configure a new mapper`.

- Name: `audience-iam-api`.
- Included Client Audience: `vacío`.
- Included Custom Audience: `iam-api`.
- Add to ID token: `Off`.
- Add to access token: `On`.
- Add to lightweight access token: `Off`.
- Add to token introspection: `On`.

<img width="826" height="738" alt="Captura de pantalla 2026-09-29 135555" src="https://github.com/user-attachments/assets/b85ecc61-1668-4eed-b004-7c53867ee9a2" />

Los dos campos de audiencia se excluyen. El primero es un desplegable con los clientes del realm y el segundo es texto libre, que es el que necesitas, porque tu API no está registrada como cliente: no pide tokens, solo los recibe.

### c) Asignar el ámbito al cliente

<img width="767" height="432" alt="Captura de pantalla 2026-09-29 135734" src="https://github.com/user-attachments/assets/48b60759-adb1-41d8-8c53-ac3529f63096" />

Clients → spa-web → pestaña Client scopes → botón Add client scope

<img width="835" height="475" alt="Captura de pantalla 2026-09-29 135808" src="https://github.com/user-attachments/assets/16ad5fe8-0011-4d28-817d-eebb06f66a5b" />

Marcamos `iam-api` y lo añades como `Default`.

<img width="566" height="82" alt="imagen" src="https://github.com/user-attachments/assets/4ef31308-b81c-41e7-a870-1c686a7de1b6" />

Este paso es imprescindible y no es el mismo que el Type Default del punto a). Aquel solo dice qué ámbitos reciben los clientes nuevos, y spa-web ya existía.

### d) Crear el rol

Realm roles → Create role.

<img width="702" height="285" alt="imagen" src="https://github.com/user-attachments/assets/1cf837c4-c962-46ce-b863-dbcacddd64de" />

nombre auditor, descripción la que sea:

<img width="736" height="377" alt="imagen" src="https://github.com/user-attachments/assets/245ffcca-8398-42c2-a0ac-82f4d704a5b5" />

### e) Asignar el rol al usuario

Users → cesar23

<img width="667" height="386" alt="imagen" src="https://github.com/user-attachments/assets/7447ed3e-c8ef-4ee0-a626-4e75c0eb2192" />

Pestaña Role mapping → Assign role. 

<img width="922" height="366" alt="imagen" src="https://github.com/user-attachments/assets/cf48b43d-17a9-4166-88d2-e0b7ac10e908" />

Cambia el filtro a Realm roles para que aparezca auditor, 

<img width="266" height="137" alt="imagen" src="https://github.com/user-attachments/assets/65248e50-0443-41d3-b52c-cc5d04a3b2c1" />

márcalo y asigna.

<img width="567" height="510" alt="imagen" src="https://github.com/user-attachments/assets/80669671-dd07-4903-b9ae-83c9521c7b6a" />

<img width="712" height="381" alt="imagen" src="https://github.com/user-attachments/assets/7e26a349-a0d6-4e0f-899d-01b424dc860a" />

