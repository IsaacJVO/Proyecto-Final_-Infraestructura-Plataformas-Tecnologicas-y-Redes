


# 📝 INFORME FINAL DEL PROYECTO SIS313  
### Balanceador de Carga y Replicación de Bases de Datos

---

## 1. Integrantes del equipo

- **Salinas Vilar Maria Luciana** (App Server 2)  
- **Vargas Oropeza Isaac Joseph** (App Server 1)

---

## 2. Objetivo del Proyecto

Implementar una arquitectura distribuida compuesta por:

- Un balanceador de carga con **NGINX**.  
- Dos servidores de aplicaciones **Node.js** con **Express** y **MySQL**.  
- Dos bases de datos **MariaDB** en modo maestro-esclavo (replicación).

---

## 3. Configuración de Red

- Red inicial: `192.168.1.0/24` → luego: `192.168.174.0/24`

### IPs Estáticas:

- 🧭 Proxy Server (NGINX): `192.168.174.200`  
- 🖥️ App Server 1: `192.168.174.201`  
- 🖥️ App Server 2: `192.168.174.202`  
- 🗄️ BD Maestro (bdserver1): `192.168.174.103`  
- 🗄️ BD Esclavo (bdserver2): `192.168.174.104`

---

## 4. Configuración de App Servers

- **Sistema operativo**: Ubuntu  
- **Tecnologías**: Node.js y Express instalados con `nvm` y `npm`  
- CRUD de `libros` con endpoints: **GET**, **POST**, **PUT**, **DELETE**

### Comandos para levantar el servidor

```bash
npm install express body-parser mysql2
node index.js
```



## Configuración de firewall

```bash
sudo ufw allow 3000
sudo ufw allow ssh
```


## 5. Balanceador de Carga con NGINX (Proxy Server)

- NGINX instalado
- Archivo: `/etc/nginx/sites-available/default`
```bash
upstream app_servers {
    server 192.168.174.201:3000;
    server 192.168.174.202:3000;
}

server {
    listen 80;

    location / {
        proxy_pass http://app_servers;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}

```


## 6. Configuración de Base de Datos Maestra (bdserver1)

- MariaDB instalado

Configuración en `/etc/mysql/mariadb.conf.d/50-server.cnf`

```bash
bind-address = 192.168.174.103
server-id = 1
log_bin = /var/log/mysql/mysql-bin.log
binlog_do_db = crud_db

```


##  7. Configuración de Base de Datos Esclava (bdserver2)

IP: `192.168.174.104`

```bash
bind-address = 192.168.174.104
server-id = 2

```

Se importó `crud_db.sql` desde el maestro


#### Configuración de replicación:
```bash
CHANGE MASTER TO
  MASTER_HOST='192.168.174.103',
  MASTER_USER='replicador',
  MASTER_PASSWORD='2005',
  MASTER_LOG_FILE='mysql-bin.000011',
  MASTER_LOG_POS=3402;

START SLAVE;

```

### Verificacion

```bash
SHOW SLAVE STATUS\G

```


### 8. Prueba de Replicación

En el maestro (`bdserver1`): 
```bash
INSERT INTO libros (titulo, autor, anio) VALUES ('Libro replicado', 'Maestro', 2025);

```

En el esclavo (`bdserver2`): 

```bash
SELECT * FROM libros;
```

## 9. Conclusiones

- El balanceador reparte correctamente las peticiones entre ambos servidores.
    
- La replicación maestro-esclavo funciona correctamente.
    
- Se logró una arquitectura distribuida, funcional y escalable.