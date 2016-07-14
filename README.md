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
Si no está instalado ya en la maquina local
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
- Escribir los siguientes comandos SQL para crear el usuario odoo (opcional, vamos a usar javierjcf):
```
$  CREATE USER javierjcf PASSWORD 'odoo' SUPERUSER;
$  \q
```
- Escribir los siguientes comandos SQL para crear el usuario javierjcf que usaremos:
```
$  CREATE USER javierjcf PASSWORD 'javierjcf' CREATEDB;
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
- Ahora procedemos a crear nuestro entorno virtual (Añadiendo -c <configuracion_elegida> se le puede pasar archivo de configuración propio)
- Es recomendable duplicar los archivos odoo.cfg y buildout.cfg a devel_odoo.cfg y devel_buildout.cfg. En el devel_buildout cambiamos la referencia de odoo.cfg a devel_odoo.cfg. (En este proyescto esta preparado para usarlo con buildout.cfg general)
```
$ sandbox/bin/python bootstrap.py -c buildout.cfg
```
- Ejecutar Buildout (con -c <configuracion_elegida> se le puede pasar el buildout.cf, que se coje por defecto o uno propio).
- Es posible que si se usa el print server  se deba instalar el paquete libcups2-dev, sino puede fallar el buildout con el paquete pycups
```
$ sudo apt-get install libcups2-dev
$ bin/buildout -c buildout.cfg
```

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