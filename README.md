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

Cambiamos el filtro a `Realm roles` para que aparezca auditor.

<img width="266" height="137" alt="imagen" src="https://github.com/user-attachments/assets/65248e50-0443-41d3-b52c-cc5d04a3b2c1" />

Marcamos y asignamos.

<img width="567" height="510" alt="imagen" src="https://github.com/user-attachments/assets/80669671-dd07-4903-b9ae-83c9521c7b6a" />

<img width="712" height="381" alt="imagen" src="https://github.com/user-attachments/assets/7e26a349-a0d6-4e0f-899d-01b424dc860a" />


### f) Comprobación antes de crear la API.

Pedimos un token nuevo en Postman y lo ponemos en el comando:

<img width="1477" height="777" alt="imagen" src="https://github.com/user-attachments/assets/23a74809-fe61-4b1e-a5ce-dae4094c4123" />

<img width="996" height="560" alt="imagen" src="https://github.com/user-attachments/assets/8436b046-f397-4164-8504-6cd773cdfe55" />

y lo ponemos en el comando:

```
AT='PEGA_EL_ACCESS_TOKEN'
```

Ahora decodificamos el token:

```
echo $AT | cut -d. -f2 | base64 -d 2>/dev/null | jq '{aud, scope, roles: .realm_access.roles}'
```

Las tres cosas que miro son estas.

El aud incluye iam-api, que es lo que confirma que el mapeador de audiencia está haciendo su trabajo. La API va a exigir ese valor, así que sin él rechazaría el token por mucho que lo demás estuviera bien.

El scope termina en iam-api, y eso confirma otra cosa distinta. Dice que el ámbito se le está aplicando al cliente spa-web. Van juntos pero no son lo mismo, y conviene no confundirlos: el scope demuestra que el ámbito llegó, y el aud demuestra que el mapeador que vive dentro de ese ámbito se ejecutó. Si viera iam-api en el scope pero no en el aud, sabría que el ámbito está bien asignado y lo que falta es el mapeador.

Y en realm_access.roles aparece auditor, que es la asignación que le hice al usuario. Ese rol todavía no sirve para nada, pero es lo que me va a permitir provocar un 403 más adelante, con un token perfectamente válido al que la API le deniega el acceso por no tener el permiso necesario.

Con esas tres confirmaciones, Keycloak queda listo y ya no vuelvo a tocarlo.

## 3.3 La API sin seguridad.

En ~/iam-api, con el entorno virtual activado, crea main.py con esto.

```
sudo nano main.py
```

```
from fastapi import FastAPI

app = FastAPI(title="IAM resource server", version="0.1.0")


@app.get("/publico")
def publico():
    return {"mensaje": "Este endpoint no exige token"}
```

<img width="747" height="262" alt="imagen" src="https://github.com/user-attachments/assets/6e341d37-6c43-4e5f-8992-88b63e9aa4f7" />

Arrancamos el servidor:

```
uvicorn main:app --reload --port 8000
```

main:app le dice a uvicorn que busque el objeto app dentro del fichero main.py. El --reload hace que se reinicie sola cada vez que guardes cambios, cómodo mientras desarrollas y desaconsejado en producción. Y el puerto 8000 es para no chocar con el 8080 de Keycloak.

Uvicorn se queda ocupando esa terminal, así que abrimos otra para probar:

```
curl -s http://localhost:8000/publico | jq
```

<img width="802" height="547" alt="imagen" src="https://github.com/user-attachments/assets/fd063558-3dc1-496e-85e8-4e97f7607e97" />

Entra también a http://localhost:8000/docs desde el navegador. FastAPI genera esa documentación leyendo tu propio código, sin que escribas nada aparte. Es el contrato de la API publicándose solo, y encaja con la definición de API que diste en la práctica anterior.

<img width="967" height="357" alt="imagen" src="https://github.com/user-attachments/assets/6bb53732-7884-42ef-9960-35e2f1de05ce" />

Si vemos los logs del uvicorn:

<img width="822" height="482" alt="imagen" src="https://github.com/user-attachments/assets/151b0a1d-9832-4941-9b23-ace4e243fb45" />

- Vemos que he intentado de manera errónea buscar una página que no existe, "documentation" y por eso aparecen errores 404.

## 3.4 Validar el Token.

Ahora vamos a reemplazar el contenido completo del "main.py".

Lo primero cancelamos la escucha del uvicorn. (CTRL + C).

```
truncate -s 0 main.py
```

```
sudo nano main.py
```

```
from fastapi import FastAPI, Header, HTTPException
import jwt
from jwt import PyJWKClient

KEYCLOAK_URL = "http://localhost:8080"
REALM = "lab-iam"
ISSUER = f"{KEYCLOAK_URL}/realms/{REALM}"
JWKS_URL = f"{ISSUER}/protocol/openid-connect/certs"
AUDIENCIA = "iam-api"

app = FastAPI(title="IAM resource server", version="0.2.0")

jwks_client = PyJWKClient(JWKS_URL)


def validar_token(authorization):
    if not authorization:
        raise HTTPException(status_code=401, detail="Falta la cabecera Authorization")

    partes = authorization.split()
    if len(partes) != 2 or partes[0].lower() != "bearer":
        raise HTTPException(status_code=401, detail="Formato esperado: Bearer <token>")

    token = partes[1]

    try:
        clave = jwks_client.get_signing_key_from_jwt(token)
        datos = jwt.decode(
            token,
            clave.key,
            algorithms=["RS256"],
            audience=AUDIENCIA,
            issuer=ISSUER,
        )
    except jwt.ExpiredSignatureError:
        raise HTTPException(status_code=401, detail="El token ha caducado")
    except jwt.InvalidAudienceError:
        raise HTTPException(status_code=401, detail=f"El token no va dirigido a {AUDIENCIA}")
    except jwt.InvalidIssuerError:
        raise HTTPException(status_code=401, detail="El emisor del token no es el esperado")
    except jwt.InvalidSignatureError:
        raise HTTPException(status_code=401, detail="La firma del token no es valida")
    except jwt.PyJWTError as e:
        raise HTTPException(status_code=401, detail=f"Token invalido ({type(e).__name__})")

    return datos


@app.get("/publico")
def publico():
    return {"mensaje": "Este endpoint no exige token"}


@app.get("/yo")
def yo(authorization: str = Header(default=None)):
    datos = validar_token(authorization)
    return {
        "sub": datos.get("sub"),
        "usuario": datos.get("preferred_username"),
        "emisor": datos.get("iss"),
        "audiencia": datos.get("aud"),
        "roles": datos.get("realm_access", {}).get("roles", []),
        "caduca": datos.get("exp"),
    }
```

Qué hace cada pieza

PyJWKClient(JWKS_URL) se crea una sola vez al arrancar. Descarga las claves públicas del realm y las cachea en memoria, y no consulta nada hasta que llega el primer token.

get_signing_key_from_jwt(token) lee el kid de la cabecera del token y busca la clave que le corresponde. Si no la tiene cacheada, vuelve a consultar el JWKS. Ese mecanismo es lo que hace que una rotación de claves en Keycloak no rompa la API.

jwt.decode es donde ocurre la validación de verdad, y hace cuatro comprobaciones de una vez. Verifica la firma con la clave que recibe, comprueba que aud incluya iam-api, comprueba que iss sea exactamente el esperado y comprueba que no haya caducado. Cada fallo lanza una excepción distinta, y por eso las capturo por separado.

El parámetro algorithms=["RS256"] no es opcional ni decorativo. Sin esa lista explícita, alguien podría presentar un token cuya cabecera diga que va firmado con none o con un algoritmo simétrico, y engañar a la librería para que lo acepte. Es una familia de ataques conocida contra implementaciones de JWT, y la defensa consiste en que sea la aplicación la que imponga qué algoritmos admite, en lugar de fiarse de lo que diga el propio token.

Volvemos a generar un token nuevo:

<img width="1185" height="707" alt="imagen" src="https://github.com/user-attachments/assets/f5890bb8-1d96-40d6-a7e0-7caad72375c6" />

Escuchamos:

```
uvicorn main:app --reload --port 8000
```

Ponemos el token nuevo:

```
AT='PEGA_UN_TOKEN_NUEVO'
curl -s http://localhost:8000/yo -H "Authorization: Bearer $AT" | jq
```

<img width="797" height="556" alt="imagen" src="https://github.com/user-attachments/assets/b79be279-e275-4509-9890-454f193ba4da" />

Como vemos funciona. Si esperamos el suficiente tiempo, dice y nos responde con que el Token caduca:

<img width="807" height="136" alt="imagen" src="https://github.com/user-attachments/assets/24f3b945-e6b2-4391-b536-a914b46fc15e" />

## 3.5 Autorizar por rol.

De nuevo, si revisamos los logs:

<img width="792" height="352" alt="imagen" src="https://github.com/user-attachments/assets/86b3d415-b0bc-47fd-aa27-cb67bc3c6a01" />

Estos dos intentos y 401, son de que el token caducó.

Dejamos de escuchar y volvemos a cambiar el "main.py".

```
truncate -s 0 main.py
```

```
nano main.py
```

```
from fastapi import FastAPI, Header, HTTPException
import jwt
from jwt import PyJWKClient

KEYCLOAK_URL = "http://localhost:8080"
REALM = "lab-iam"
ISSUER = f"{KEYCLOAK_URL}/realms/{REALM}"
JWKS_URL = f"{ISSUER}/protocol/openid-connect/certs"
AUDIENCIA = "iam-api"

app = FastAPI(title="IAM resource server", version="0.3.0")

jwks_client = PyJWKClient(JWKS_URL)


def validar_token(authorization):
    if not authorization:
        raise HTTPException(status_code=401, detail="Falta la cabecera Authorization")

    partes = authorization.split()
    if len(partes) != 2 or partes[0].lower() != "bearer":
        raise HTTPException(status_code=401, detail="Formato esperado: Bearer <token>")

    token = partes[1]

    try:
        clave = jwks_client.get_signing_key_from_jwt(token)
        datos = jwt.decode(
            token,
            clave.key,
            algorithms=["RS256"],
            audience=AUDIENCIA,
            issuer=ISSUER,
        )
    except jwt.ExpiredSignatureError:
        raise HTTPException(status_code=401, detail="El token ha caducado")
    except jwt.InvalidAudienceError:
        raise HTTPException(status_code=401, detail=f"El token no va dirigido a {AUDIENCIA}")
    except jwt.InvalidIssuerError:
        raise HTTPException(status_code=401, detail="El emisor del token no es el esperado")
    except jwt.InvalidSignatureError:
        raise HTTPException(status_code=401, detail="La firma del token no es valida")
    except jwt.PyJWTError as e:
        raise HTTPException(status_code=401, detail=f"Token invalido ({type(e).__name__})")

    return datos


def exigir_rol(datos, rol):
    roles = datos.get("realm_access", {}).get("roles", [])
    if rol not in roles:
        raise HTTPException(status_code=403, detail=f"Hace falta el rol '{rol}'")


@app.get("/publico")
def publico():
    return {"mensaje": "Este endpoint no exige token"}


@app.get("/yo")
def yo(authorization: str = Header(default=None)):
    datos = validar_token(authorization)
    return {
        "sub": datos.get("sub"),
        "usuario": datos.get("preferred_username"),
        "emisor": datos.get("iss"),
        "audiencia": datos.get("aud"),
        "roles": datos.get("realm_access", {}).get("roles", []),
        "caduca": datos.get("exp"),
    }


@app.get("/informes")
def informes(authorization: str = Header(default=None)):
    datos = validar_token(authorization)
    exigir_rol(datos, "auditor")
    return {
        "informes": ["cierre-mensual", "accesos-privilegiados"],
        "solicitado_por": datos.get("preferred_username"),
    }
```

Fíjate en el orden dentro del endpoint, porque ahí está toda la teoría del apartado de autenticación y autorización hecha código. Primero validar_token, que establece quién eres y devuelve 401 si no puede. Después exigir_rol, que decide qué puedes hacer y devuelve 403 si no te corresponde. Son dos pasos separados y en ese orden, nunca al revés.

Escuchamos:

```
uvicorn main:app --reload --port 800
```

Generamos otro token, lo almacenamos:

```
AT='AQUI EL ACCESS TOKEN'
```

Probamos con el rol puesto.

```
curl -s http://localhost:8000/informes -H "Authorization: Bearer $AT" | jq
```

Probamos con el rol puesto.

<img width="805" height="757" alt="imagen" src="https://github.com/user-attachments/assets/94b8be16-2618-46b8-8615-f5ee5bc1b242" />

Debería darte los informes. Si tardamos mucho, caduca y toca hacer el token de nuevo.

Y ahora nos quitamos el rol: 

Keycloak → Users → cesar23 → pestaña Role mapping

<img width="806" height="217" alt="imagen" src="https://github.com/user-attachments/assets/ae34b993-1ac5-4258-9b12-67280ff4632b" />

Seleccionamos `auditor` y pulsas `Unassign`.

Ya no lo es:

<img width="940" height="481" alt="Captura de pantalla 2026-10-01 170623" src="https://github.com/user-attachments/assets/b07228b0-e5a3-46f0-8cb0-7349aef127a6" />

De nuevo, el token si o si, tiene que estar caducado entonces generamos uno.

<img width="807" height="496" alt="imagen" src="https://github.com/user-attachments/assets/e6042cb2-9cef-4b33-bdd2-b836ae218ea7" />

Sin embargo, vamos a ver para lo que sí estamos autorizados, para ello rápidamente ejecutamos:

```
curl -s http://localhost:8000/yo -H "Authorization: Bearer $AT" | jq
```

<img width="800" height="587" alt="imagen" src="https://github.com/user-attachments/assets/1e500fa2-a247-4832-8220-2a5c4b6e917a" />

Y funciona. Debe devolver 200. Mismo token, mismo servidor, dos endpoints y dos respuestas distintas. Eso es autorización.

## 3.6 El resto de los fallos

> [!NOTE]
> Si te caduca el token, simplemente genera uno nuevo para probar estas cosas.
> 

Para este último apartado, vamos a provocar situaciones/errores:

### A) Sin cabecera.

```
curl -i -s http://localhost:8000/yo | head -1
```
<pre>
HTTP/1.1 401 Unauthorized
</pre>

### B) Cabecera mal formada.

```
curl -i -s http://localhost:8000/yo -H "Authorization: $AT" | head -1
```
<pre>
HTTP/1.1 401 Unauthorized
</pre>

### C) Firma manipulada.

```
H=$(echo $AT | cut -d. -f1); P=$(echo $AT | cut -d. -f2); S=$(echo $AT | cut -d. -f3)
NEWP=$(echo $P | tr '_-' '/+' | base64 -d 2>/dev/null | sed 's/"acr":"1"/"acr":"9"/' | base64 -w0 | tr '/+' '_-' | tr -d '=')
curl -s http://localhost:8000/yo -H "Authorization: Bearer $H.$NEWP.$S" | jq
```
<pre>
{
  "sub": "920ba24c-b418-4c90-99af-48c49078b7a1",
  "usuario": "cesar23",
  "emisor": "http://localhost:8080/realms/lab-iam",
  "audiencia": [
    "iam-api",
    "account"
  ],
  "roles": [
    "offline_access",
    "default-roles-lab-iam",
    "uma_authorization"
  ],
  "caduca": 1790868192
}
</pre>

### D) Token caducado: espera a que pase el minuto y repite.

```
curl -s http://localhost:8000/yo -H "Authorization: Bearer $AT" | jq
```

<pre>
{
  "sub": "920ba24c-b418-4c90-99af-48c49078b7a1",
  "usuario": "cesar23",
  "emisor": "http://localhost:8080/realms/lab-iam",
  "audiencia": [
    "iam-api",
    "account"
  ],
  "roles": [
    "offline_access",
    "default-roles-lab-iam",
    "uma_authorization"
  ],
  "caduca": 1790868192
}
</pre>
