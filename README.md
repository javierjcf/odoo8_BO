Este repositorio contiene el Buildout del proyecto.
No se incluye postgres en el entorno virtual, pero si se presupone instalado en el sistema, asi como un usuario odoo con capacidad de superusuario en postgres con contraseña postgres.
Se creará una base de datos por defecto llamada odoo

# BUILDOUT PARA PRUEBAS
- Las intrucciones de los siguientes apartados no tienen que ser 100 para este buildout
- Se prescinde de crear devel_buildout y devel_odoo.
- base-odoo-pg.conf cuenta con partes para postgres y supervisor
- buildout.cfg modificado desde base para no tener postgres ni supervisor.
- Desarrollos propios en carpeta project-addons.
- Se requiere tener postgres instalado.
- Usuario y contraseña para postgres: javierjcf (Tener en cuenta al crear usuario con instrucciones de abajo)

# CARACTERISTICAS
- Odoo 8.0 commit específico en el base, Supervisord 3.0
- Supervisor ejecuta PostgreSQL en el entorno virtual para este proyecto
- PostgreSQL debe estar instalado en el sistema
- Debe haber creado un superusuario odoo en postgres con contraseña odoo (se puede modificar), se detalla la instalación y configuración de postgres en la máquina local.
- Buildout crea cron para iniciar Supervisord después de reiniciar (se han de tener permisos de escritura en la propia carpeta)
- Si existe un archivo dump.sql, el sistema generará la base de datos con ese dump (probar)
- Si existe  un archivo frozen.cfg es el que se debeía usar ya que contiene las revisiones aprobadas (probar)

# INSTALACIÓN DEPENDENCIAS
En caso de no haberse hecho antes en la máquina en la que se vaya a realizar:
- Añadir el repo Anybox a /etc/apt/sources.list:
```
$ deb http://apt.anybox.fr/openerp common main
```
- Si se quiere añadir la firma. Esta a veces tarda mucho tiempo o incluso da time out. Es opcional meterlo
```
$ sudo apt-key adv --keyserver hkp://subkeys.pgp.net --recv-keys 0xE38CEB07
```
- Actualizar e instalar
```
$ sudo apt-get update
$ sudo apt-get install openerp-server-system-build-deps
```
- Para poder compilar e instalar postgres (debemos valorar si queremos hacerlo siempre), es necesario instalar el siguiente paquete (no es la solución ideal, debería poder hacerlo el propio buildout, pero de momento queda así, solo en caso de que se use postgres en el buildout, no necesario en este proyecto)
```
$ sudo apt-get install libreadline-dev
```
- Instalar paquete python de entorno virtual
```
$ sudo apt-get install python-virtualenv
```

# INSTALACIÓN Y CONFIGURACION DE POSTGRES
- Instalar versión de postgres deseada (se recomienda mayor que 9.1):
```
$  sudo apt-get install postgresql
```
- Cambiar a usuario postgres :
```
$  sudo su postgres
```
- Acceder a la base de datos :
```
$  psql template1
```
- Escribir los siguientes comandos SQL para crear el usuario odoo:
```
$  CREATE USER odoo PASSWORD 'odoo' SUPERUSER;
$  \q
```
- Salir de la sesión de postgres:
```
$  exit
```
- Cambiar el método de autentificación a md5 editando el fichero de configuración de postgres.
```
$  sudo nano /etc/postgresql/[version]/main/pg_hba.conf
```
- Al fondo del fichero localizar las lineas de abajo y cambiar el método de autentificacion a md5 (a veces viene peer), debe quedar:
```
# TYPE  DATABASE        USER            ADDRESS                 METHOD
# "local" is for Unix domain socket connections only
local   all             all                                     md5
```
- Reiniciar postgres
```
 sudo /etc/init.d/postgresql restart
```

# DESCARGA Y EJECUCIÓN PROYECTO
- Descargar el  repositorio del buildout de Proyecto(<ubicacion_local_repo> será la carpeta que lo contenga):
```
$  git clone git@github.com:Comunitea/CMNT_00040_2016_ELN.git <ubicacion_local_repo>
```
- Crear un virtualenv dentro de la carpeta del respositorio. Esto podría ser opcional, obligatorio para desarrollo o servidor de pruebas, tal vez podríamos no hacerlo para un despliegue en producción.
```
$ cd <ubicacion_local_repo>
$ virtualenv sandbox --no-setuptools
```
- Crear la carpeta eggs:
```
$ mkdir eggs
```
- Crear la carpeta odoo_reports que contendrá todos los repositorios necesarios para el proyecto como los específicos, localización española, módulos de account, etc...
```
$ mkdir odoo_repos
```
- Ahora procedemos a crear nuestro entorno virtual (Añadiendo -c <configuracion_elegida> se le puede pasar archivo de configuración propio)
- Es recomendable duplicar los archivos odoo.cfg y buildout.cfg a devel_odoo.cfg y devel_buildout.cfg. En el devel_buildout cambiamos la referencia de odoo.cfg a devel_odoo.cfg.
```
$ sandbox/bin/python bootstrap.py
```
- Ejecutar Buildout (con -c <configuracion_elegida> se le puede pasar el buildout.cf, que se coje por defecto o uno propio).
- Es posible que si se usa el print server  se deba instalar el paquete libcups2-dev, sino puede fallar el buildout con el paquete pycups
```
$ sudo apt-get install libcups2-dev
$ bin/buildout -c devel_buildout.cfg
```
- Lo último que hace es intentar crear la base de datos odoo si no existiese. Puede ser necesario introducir el password, que es odoo
- Opcionalmente podemos lanzar supervisor
```
$ bin/supervisord
```
- Urls
- Supervisor : http://localhost:9003
- Odoo: http://localhost:9069
        user: admin//pass: admin

## Configurar OpenERP
Archivo de configuración: etc/openerp.cfg, si sequieren cambiar opciones en  openerp.cfg, no se debe editar el fichero,
si no añadirlas a la sección [openerp] del buildout.cfg
y establecer esas opciones .'add_option' = value, donde 'add_option'  y ejecutar buildout otra vez.

Por ejmplo: cambiar el nivel de logging de OpenERP
```
'buildout.cfg'
...
[openerp]
options.log_handler = [':ERROR']
...
```

Si se quiere ejecutar más de una instancia de OpenERP, se deben cambiar los puertos,
please change ports:
```
openerp_xmlrpc_port = 8069  (8069 default openerp)
openerp_xmlrpcs_port = 8071 (8071 default openerp)
supervisor_port = 9002      (9001 default supervisord)
postgres_port = 5434        (5432 default postgres)
```

# GESTION BASES DE DATOS
- Para manejar las bases de datos con postgres, se usan los comandos del sistema añadiendo -U odoo.
- Ejemplo importar base de datos existente
```
createdb -U odoo nueva_bd
psql -U nueva_bd < ruta/del/backup.sql
```
- Ejemplo backup base de datos:
```
pg_dump -U odoo bd_existente > ruta/del/backup.sql
```
# Buildout base para proyectos con Odoo y PostgreSQL
Odoo 8.0 en el base, PostgreSQL 9.5.2 y Supervisord 3.0
- Buildout crea cron para iniciar Supervisord después de reiniciar (esto no lo he probado)
- Supervisor ejecuta PostgreSQL, más info http://supervisord.org/
- También ejecuta la instancia de PostgreSQL
- Si existe un archivo dump.sql, el sistema generará la base de datos con ese dump
- Si existe  un archivo frozen.cfg es el que se debeía usar ya que contiene las revisiones aprobadas
- PostgreSQL se compila y corre bajo el usuario user (no es necesario loguearse como root), se habilita al autentificación "trust" para conexiones locales. Más info en more http://www.postgresql.org/docs/9.3/static/auth-methods.html
- Existen plantillas para los archivo de configuración de Postgres que se pueden modificar para cada proyecto.


# Uso (adaptado)
En caso de no haberse hecho antes en la máquina en la que se vaya a realizar, instalar las dependencias que mar Anybox
- Añadir el repo a /etc/apt/sources.list:
```
$ deb http://apt.anybox.fr/openerp common main
```
- Si se quiere añadir la firma. Esta a veces tarda mucho tiempo o incluso da time out. Es opcional meterlo
```
$ sudo apt-key adv --keyserver hkp://subkeys.pgp.net --recv-keys 0xE38CEB07
```
- Actualizar e instalar
```
$ sudo apt-get update
$ sudo apt-get install openerp-server-system-build-deps
```
- Para poder compilar e instalar postgres
```
$ sudo apt-get install libreadline-dev
```
- Crear un virtualenv dentro de la carpeta del respositorio. Esto podría ser opcional, obligatorio para desarrollo o servidor de pruebas, tal vez podríamos no hacerlo para un despliegue en producción. Si no está instalado, instalar el paquete de virtualenv. Es necesario tener la versión que se instala con easy_install o con pip, desinstalar el paquete python-virtualenv si fuera necesario e instalarlo con easy_install
```
$ sudo easy_install virtualenv
$ virtualenv sandbox --no-setuptools
```
- Ahora procedemos a ejecutar el buildout en nuestro entorno virtual
```
$ sandbox/bin/python bootstrap.py -c [archivo_buildout]
```
- Lanzar buildout (el -c [archivo_buildout] se usa cuando no tiene el nombre por defecto buildout.cfg)
```
$ bin/buildout -c [archivo_buildout]
```

- Puede que de error, hay que lanzar el supervisor y volver a hacer bin/buildout:
```
$ bin/supervisord
$ bin/buildout -c [archivo_buildout]
```
- Conectarse al supervisor con localhost:9002
- Si fuera necesario hacer update all, se puede parar desde el supervisor y en la consola hacer:
```
$ cd bin
$ ./upgrade_openerp
```
- odoo se lanza en el puerto 9069 (se pude configurar en otro)


## Configurar Odoo
Archivo de configuración: etc/openerp.cfg, si sequieren cambiar opciones en  openerp.cfg, no se debe editar el fichero,
si no añadirlas a la sección [openerp] del buildout.cfg
y establecer esas opciones .'add_option' = value, donde 'add_option'  y ejecutar buildout otra vez.

Por ejmplo: cambiar el nivel de logging de OpenERP
```
'buildout.cfg'
...
[openerp]
options.log_handler = [':ERROR']
...
```

Si se quiere ejecutar más de una instancia de OpenERP, se deben cambiar los puertos,
please change ports:
```
openerp_xmlrpc_port = 9069  (8069 default openerp)
openerp_xmlrpcs_port = 9071 (8071 default openerp)
supervisor_port = 9002      (9001 default supervisord)
postgres_port = 5434        (5432 default postgres)
```

# Contributors

## Creators

Rastislav Kober, http://www.kybi.sk

