# Instal·lació de MariaDB amb Docker

Per realitzar la instal·lació de MariaDB, necessitarem una màquina (preferiblement amb Linux) on hi tindrem un sistema de contenidors (en el meu cas Docker) per poder arrancar el SGBD ràpidament i tindre control total sobre aquest.
<p align="center">
  <img src="https://i.imgur.com/bVCKyQ8.png">
</p>

---

## 1. Instal·lació de Docker i Docker Compose

### Primer de tot, cal verificar que hem instal·lat docker correctament amb la següent comanda:

```bash
docker --version
docker-compose --version
```

<p align="center">
  <img src="https://i.imgur.com/LpE2wYT.png">
</p>

Si no els tens instal·lats, segueix les instruccions de la [documentació oficial de Docker](https://docs.docker.com/get-docker/):
  - [Docker](https://docs.docker.com/engine/install/)
  - [Docker-Compose](https://docs.docker.com/compose/install/)

---

## 2. Crearem una ruta dins del nostre servidor, que utilitzarem per poder guardar l’arxiu de Docker-Compose i altres components del SGBD.

### Crear la carpeta i accedir a ella:

```bash
Crear Carpeta → mkdir mariadb
Accedir Carpeta → cd mariadb
```

<p align="center">
  <img src="https://i.imgur.com/FVPI5vt.png">
</p>

## 3. Crear arxiu docker-compose.yml

```bash
Crear Arxiu → touch docker-compose.yml
```

<p align="center">
  <img src="https://i.imgur.com/f0RkHjY.png">
</p>

### Verificar imatge a DockerHub

Abans de modificar l’arxiu, verifiquem a la web de Docker Hub si existeix alguna imatge de MariaDB creada per un altre usuari/a:

<p align="center">
  <img src="https://i.imgur.com/OmohiiN.png">
</p>

<p align="center"><i>En el meu cas he escollit la primera, ja que és la que més descàrregues té.</i></p>

## 4. Modificar l’arxiu docker-compose per desplegar el SGBD.

```bash
Modificar arxiu → nano docker-compose.yml
```

### Afegeix el següent contingut a l'arxiu docker-compose.yml:

```yaml
version: '3.1'

services:
  mariadb:
    image: mariadb:latest
    container_name: mariadb_proves
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: P@ssword
      MYSQL_DATABASE: proves
      MYSQL_USER: llucdani
      MYSQL_PASSWORD: P@ssw0rd
    ports:
      - "3306:3306"
```

<p align="center">
  <img src="https://i.imgur.com/2YHZFxS.png">
</p>

	- Image → Nom de la imatge que hem seleccionat de dockerhub (:latest indica l’última versió).
	- container_name → Nom del nostre contenidor.
	- restart: always → Arranca el contenidor automàticament, si hi ha un error es reinicia.
	- environment → Variables d’entorn predefinides de MariaDB
	- port → Port per on escolta la BBDD.

---

## 5. Arrancar el nostre contenidor utilitzant docker-compose.

```bash
Arrancar Contenidor → docker-compose up -d
```

<p align="center">
  <img src="https://i.imgur.com/uD83W4d.png">
</p>

<p align="center"><i>Començarà a descarregar la imatge per primer cop…</i></p>

<p align="center">
  <img src="https://i.imgur.com/SZh3kXA.png">
</p>

### Verificar que el contedidor ha arrancat correctament:
```bash
docker ps
```
<p align="center">
  <img src="https://i.imgur.com/83tPCHS.png">
</p>

### Si no apareix, pots revisar els logs:
```bash
docker logs [nom_contenidor]
```

---

## 6. Accedir dins del contenidor i identificar l’arxiu de configuració

### Per accedir al contenidor i treballar amb MariaDB:
```bash
docker exec -it mariadb_proves bash
```
<p align="center">
  <img src="https://i.imgur.com/ycb6ryq.png">
</p>

### En el cas de MariaDB, la carpeta de configuració es troba en `/etc/mysql`

<p align="center">
  <img src="https://i.imgur.com/7dy7nuH.png">
</p>

---

## 7. Mapejar la carpeta de configuració a la màquina host.

### Cal mapejar la carpeta de dades i configuració de MariaDB, ja que quan reiniciem el contenidor o la màquina ens interessa que no s’esborri res, ho podem fer modificant l’arxiu de docker-compose.

Però primer de tot, hem de copiar les carpetes des del contenidor a la màquina amfitrió i parar el contenidor.
- **Arxius Configuració:** `/etc/mysql`
- **Arxius Dades:** `/var/lib/mysql`

Copiar les dades del contenidor al teu host:
```bash
docker cp mariadb_container:/etc/mysql ./config
docker cp mariadb_container:/var/lib/mysql ./data
```
<p align="center">
  <img src="https://i.imgur.com/iP8eKEq.png">
</p>
<p align="center">
  <img src="https://i.imgur.com/cdsUBI1.png">
</p>

Parar el contedidor:
```bash
docker-compose down
```
<p align="center">
  <img src="https://i.imgur.com/nFFnMsD.png">
</p>

### Un cop ja tenim les carpetes copiades, hem de modificar la configuració de l’arxiu docker-compose.yml i mapejar els volums amb la següent configuració:
```yaml
version: '3.1'

services:
  mariadb:
    image: mariadb:latest
    container_name: mariadb_proves
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: P@ssword
      MYSQL_DATABASE: proves
      MYSQL_USER: llucdani
      MYSQL_PASSWORD: P@ssw0rd
    ports:
      - "3306:3306"
      - volumes:
        - ./configuracio/mysql:/etc/mysql
        - ./mysql:var/lib/mysql
```

<p align="center">
  <img src="https://i.imgur.com/k1Hljls.png">
</p>

### Ara que ja tenim persistència, podem arrancar el contenidor.
<p align="center">
  <img src="https://i.imgur.com/G7l0Knj.png">
</p>

<p align="center"><i>Ara ja estem preparats per connectar-nos a la BBDD.</i></p>

---

## 8. Connectar-se a la BBDD

Per realitzar la connexió a la nostra BBDD, podem utilitzar programes com:
- [MySQL Workbench](https://www.mysql.com/products/workbench/)
- [HeidiSQL](https://www.heidisql.com/)
- [DBeaver](https://dbeaver.io/)
- [Navicat](https://www.navicat.com/) *[pagament]*

### En el meu cas, faré una demostració amb HeidiSQL, ja que m’ha semblat fàcil e intuïtiu.

Podem descarregar el software de la [web oficial](https://www.heidisql.com/download.php):
<p align="center">
  <img src="https://i.imgur.com/JOFzjOD.png">
</p>

Arrancar el programa i realitzar la connexió:
- **Host:** `IP_SERVIDOR` 
- **Usuari:** `llucdani` _# Usuari que hem introduït a l'arxiu docker-compose.yml_
- **Contrasenya:** `P@ssw0rd` _# Password que hem introduït a l'arxiu docker-compose.yml_
- **Port:** `3306` _# Port que hem introduït a l'arxiu docker-compose.yml_
<p align="center">
  <img src="https://i.imgur.com/hP4Ky8d.png">
</p>

Si hem realitzat tots els passos correctament., ens podrem connectar sense cap problema a la BBDD.
<p align="center">
  <img src="https://i.imgur.com/dzeMi0b.png">
</p>
<p align="center"><i>Podem observar la nostra BBDD inicial que hem indicat a l’arxiu de docker-compose. </i></p>

<p align="center">
  <img src="https://i.imgur.com/OI3BANs.png">
</p>
<p align="center"><i>Es pot veure que les consultes funcionen correctament.</i></p>

---

# Administració Bàsica de MariaDB

### 1. Arrencar MariaDB
```bash
docker-compose up -d
```

### 2. Verificar l'estat
```bash
docker ps
docker logs mariadb_container
```

### 3. Aturar MariaDB
```bash
docker-compose down
```

---

# Configuració avançada

### 1. Fitxer de configuració
Ubicació: `/etc/mysql/mariadb.cnf`
<p align="center">
  <img src="https://i.imgur.com/cP0phs1.png">
</p>
<p align="center"><i>Com he utilitzat docker, ho tinc mapejat a la caprpeta de mariadb que hem creat al principi.</i></p>

### 2. Canviar el port d'escolta
Si vols modificar el port, edita `docker-compose.yml` i canvia la línia:
```yaml
ports:
  - "3307:3306"
```
Després, reinicia el contenidor:
```bash
docker-compose down
docker-compose up -d
```
<p align="center">
  <img src="https://i.imgur.com/xdz7Pi7.png">
</p>

---

# Securitzar l'instal·lació

- Podem utilitzar SSL per xifrar les dades que viatgen des del client fins la BBDD combinant OpenSSL i Docker:

```bash
# Comandes per generar un certificat autofirmat 
openssl genrsa 2048 > ca-key.pem

openssl req -new -x509 -nodes -days 365 -key ca-key.pem -out ca-cert.pem

openssl req -newkey rsa:2048 -nodes -keyout server-key.pem -out server-req.pem

openssl x509 -req -in server-req.pem -days 365 -CA ca-cert.pem -CAkey ca-key.pem -set_serial 01 -out server-cert.pem
```

- Un cop generat el certificat, afegirem les següents línies a l'arxiu docker.compose.yml:

```yaml
version: "3.8"

services:
  mariadb:
    image: mariadb:latest
    container_name: mariadb_proves
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: P@ssword
      MYSQL_DATABASE: proves
      MYSQL_USER: llucdani
      MYSQL_PASSWORD: P@ssw0rd
    ports:
      - "3306:3306"
    - volumes:
      - ./configuracio/mysql:/etc/mysql
      - ./mysql:var/lib/mysql
      - ./ssl:/etc/mysql/ssl
    command: >
      --ssl-ca=/etc/mysql/ssl/ca-cert.pem
      --ssl-cert=/etc/mysql/ssl/server-cert.pem
      --ssl-key=/etc/mysql/ssl/server-key.pem
```

- Ara ja ens podrem conectar utilitzant l'opció "SSL" en HeidiSQL

<p align="center">
  <img src="https://i.imgur.com/fqjoluf.png">
</p>



