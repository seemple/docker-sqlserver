### Acerca de los permisos:

El usuario que ejecuta comandos en el contenedor NO existe en el equipo local. Debido a que el contenedor posee un volumen (un enlace entre el equipo local y el contenedor para persistir los datos al reiniciar el contenedor ) debemos darle permisos de escritura al usuario del contenedor, en el equipo local. 

### Para ello debemos:

1. Averiguar el id del usuario del contenedor: 
```docker compose exec -it db id mssql``` 
lo cual responderá: ```uid=10001(mssql) gid=0(root) groups=0(root)```

2. Brindar permisos al usuario mssql(10001) en todas las carpetas del equipo LOCAL para la escritura de la db:
```
sudo chown -R 10001 /var/lib/docker/volumes/sqlserver-docker_sqldata/
sudo chown -R 10001 /var/lib/docker/volumes/sqlserver-docker_sqllog/
```
Para averiguar la ubicación de las carpetas relacionadas con los volumenes, puedes ejecutar:
```docker volume inspect [VOLUME_NAME]```

**Luego ya podemos crear la base de datos:**
```docker compose exec -it db /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -d master```
**Opción 1:** Una vez dentro del contenedor, ejecutar el comando de T-SQL para crear la DB:
```
CREATE DATABASE Terrafertil;
GO
```
**Opción 2:** podemos crear la DB desde un sql file ( que previamente debemos copiar al contenedor )
```docker cp start.sql db-sqlserver:/var/opt/sqlserver/backup```

Y luego ejecutar el comando con la creación:
```docker compose exec -it db /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -d master -i /var/opt/sqlserver/backup/start.sql```

**Copiar DB a restaurar al contenedor:**
```docker cp TERRAFERTIL_2018.bak db-sqlserver:/var/opt/sqlserver/backup```

**Restaurar DB:**
```USE [master] RESTORE DATABASE [Terrafertil] FROM DISK = N'/var/opt/sqlserver/backup/TERRAFERTIL_2018.bak' WITH  FILE = 1, NOUNLOAD, REPLACE, STATS = 10```