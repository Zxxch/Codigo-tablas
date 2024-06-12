# TP Desarrollo de Software

## Integrantes:

- Avendaño Malvina
- Libovich Damian
- Mateos Victoria
- Juan Camila
- Palacio Joaquin
- Veri Joaquin
- Videla Milagros

### Consigna:

Integración del eCommerce con el Sistema Legacy. Toda nuestra data de inventario se encuentra en una base de datos Postgres que se encuentra en nuestro datacenter, queremos acceder a la misma pero siguiendo los estándares de la compañía.

### Modelado de la Base de Datos

| Campo       | Tipo    | Descripción                | Key | Incremental |
| ----------- | ------- | -------------------------- | --- | ----------- |
| id          | serial  | Identificador del producto | PK  | Si          |
| nombre      | varchar | Nombre del producto        | No  | No          |
| precio      | numeric | Precio del producto        | No  | No          |
| stock       | int     | Stock del producto         | No  | No          |
| descripcion | text    | Descripción del producto   | No  | No          |
| categoria   | varchar | Categoría del producto     | No  | No          |
| imagen      | text    | Imagen del producto        | No  | No          |

### Datos del producto:

- **Nombre:** Juego de utensilios
- **Precio:** $49.99
- **Stock:** 100
- **Descripción:** Herramientas esenciales de la cocina
- **Categoría:** Hogar y Cocina
- **Imagen:** https://m.media-amazon.com/images/I/61pStIrJeWL._AC_SL1500_.jpg

## Pasos seguidos

Crear la base de datos ecommerce.

```sql
CREATE DATABASE ecommerce;
```

Crear una tabla con el siguiente script:

```sql
CREATE TABLE productos (
    id SERIAL PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    precio NUMERIC(10, 2) NOT NULL,
    stock INTEGER NOT NULL,
    descripcion TEXT,
    categoria VARCHAR(50),
    imagen TEXT
);
```

Llenar la tabla con los datos dados:

```sql
INSERT INTO public.productos (nombre, precio, stock, descripcion, categoria, imagen)
VALUES ('Juego de utensilios', 49.99, 100, 'Herramientas esenciales de la cocina', 'Hogar y Cocina', 'https://m.media-amazon.com/images/I/61pStIrJeWL._AC_SL1500_.jpg');
```

Navegar en la consola hacia `cd "C:\Program Files\PostgreSQL\<versión>\bin"`  
En mi caso en _version_ iba 16

Abrir el intérprete de comandos de PostgreSQL  
`psql -U postgres`  
Va a pedir la contraseña que eligieron cuando descargaron postgres.

Crear usuario nuevo  
`CREATE USER super WITH PASSWORD 'password';`  
En este caso estamos creando el usuario _super_ con la contraseña _password_.

Darle permisos  
`ALTER USER super WITH SUPERUSER;`

Salir del intérprete  
`\q`

En `C:\Program Files\PostgreSQL\<versión>\bin`, hacer el backup de la base de datos, esto creara un archivo llamado “_backup_db.dump_” con el backup de la base de datos.  
`pg_dump -U super -d ecommerce -F c -b -v -f backup_db.dump`

Copiar el archivo **backup_db.dump** a una nueva carpeta.

En esa carpeta crear un archivo llamado **docker-compose.yml**

En ese archivo poner lo siguiente:

```yaml
version: "3"
services:
  db:
    image: postgres:latest
    container_name: postgres_container
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: ecommerce
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data
volumes:
  pgdata:
```

Dirígete a esa carpeta en la consola y levantar el contenedor  
`docker-compose up -d`

Copiar el backup en el contenedor  
`docker cp backup_db.dump postgres_container:/backup_db.dump`

Ingresar al contenedor  
`docker exec -it postgres_container bash`

Restaurar el backup  
`pg_restore -U user -d ecommerce -v /backup_db.dump`

Configurar el router para que las conexiones al puerto **5432** vayan a la pc donde está almacenado el container Docker.

Configurar el firewall de la pc para que permita accesos.

Desde otra pc en pgadmin, agregar un servidor y pasarle como parámetros la ip pública de la pc donde está almacenada el docker, el puerto (5432), el usuario (super) y la contraseña (password).
