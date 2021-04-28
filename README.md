Este documento se crea con la finalidad de poder crear una instancia del proyecto en cualquier servidor nuevo.

Git por default omite muchos archivos del versionado, esto lo hace para alivianar el tráfico y en segunda instancia para no editar archivos que NO se deben editar (vendors).

Los archivos en el repositorio son los modificados por el desarrollador.

Instalando el proyecto:

# PRE REQUISITOS: #
A continuación se presentan una lista de software mínimo necesario para levantar un entorno tanto de desarrollo como productivo.

## Servidor ##
- PHP: 7.2 (o superior)
- DB: mariadb-10.1.33 (o superior)
- RAM: 1gb (o más)

## Composer ##
(El encargado de instalar y actualizar los vendors de symfony)
```
curl -sS https://getcomposer.org/installer | php
mv composer.phar ~/.local/bin
```

## npm ##
(A.K.A. NodeJs)
```
sudo yum install nodejs
```
## Yarn ##
Lo utilizamos para realizar copias prolijas de assets internos
```
curl -o- -L https://yarnpkg.com/install.sh | bash
```

-----------------------------

## Consideraciones ##

Se debe clonar el proyecto en la carpeta de acceso publico (htdocs en xampp, var/www/html/ en alguna distribución de linux, etc...)

El proyecto clonado posee un archivo **composer.json** que posee todas las dependencias hasta el momento (paquetes externos necesarios)

Una vez clonado el proyecto, en la raiz del mismo (En este caso SignosSF) se deben instalar las dependencias con el siguiente comando

```
composer install
```
Dependiendo el sistema o el tipo de instalación, puede variar en algo como esto la ejecución (u alguna que otra variante)
```
php composer.phar install
```

Esto creará la carpeta vendor con todas las dependencias que hasta el momento existen (se irán agregando pocas más, está bastante completo el composer.json)

El siguiente paso es configurar el ambiente (enviroment), para ello sólo tenemos que editar el archivo **.env**

Este archivo tiene muchas cosas y muchas de ellas comentadas (las dejé así porque no es necesario por ahora utilizarlas)

Pero las que nos interesan son estas:
```
APP_ENV=dev
DATABASE_URL=mysql://db_user:db_password@127.0.0.1:3306/db_name
MAILER_DSN=smtp://localhost:25?encryption=ssl&auth_mode=login&username=&password=
```
La primera es el entorno, dev (desarrollo) y prod (producción), así mismo se pueden crear infinidad de otros entornos como de test, no hay limites...

La segunda es la configuración de base de datos, que son claros los requerimientos que pide.

Una vez configurado eso, vamos a crear la migración de tablas, para ello dejé preparado una serie de comandos para ejecutar en este orden:

> Crear la base de datos con el nombre que se le haya dado en la configuración del archivo .env
```
php bin/console doctrine:database:create
```
> Los comandos son cmd...bat para entorno windows cmx...sh para entorno linux, a modo de ejemplo voy a usar los de linux.

Impacta toda la estructura creada con el comando anterior en la base de datos
```
cmxMigrarTablas.sh
```
Inserta una serie de datos de prueba generados por el desarrollador.  si bien este comando NO es necesario, yo lo corro porque me crea el acceso de administrador con usuario admin, clave 123456
```
cmxCargarFixtures.sh
```

Debemos instalar las dependencias del yarn
```
yarn install
```

Asumiendo que es entorno desarrollo, se ejecuta este comando, que limpia todos los datos de cache (si hubiese) y deja la aplicación lista.
```
cmxActualizaDev.sh
```
Por último y asumiendo entorno desarrollo se ejecuta el siguiente comando que permite acceder a la aplicación desde la url: http://127.0.0.1:8000/
```
cmxStartServerDev.sh
```

## UPDATE RUTA/htaccess ##


Suponiendo que el proyecto se encuentre en /var/www/proyecto
Y que el CPanel (o VirtualHost) apunte a el directorio /var/www/proyecto

Tenemos que tener en cuenta dos cosas (dos .htaccess)

> /var/www/proyecto/.htaccess

```
<IfModule mod_rewrite.c>
RewriteEngine on
RewriteCond %{HTTP_HOST} ^proyecto.com$ [NC,OR]
RewriteCond %{HTTP_HOST} ^www.proyecto.com$
RewriteCond %{REQUEST_URI} !public/
RewriteRule (.*) /public/$1
</IfModule>
```

> /var/www/proyecto/public/.htaccess (https://gist.github.com/Guibzs/a3e0b3ea4eb00c246cda66994defd8a4)

```
<IfModule mod_rewrite.c>
    Options -MultiViews
    RewriteEngine On
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^(.*)$ index.php [QSA,L]
</IfModule>

<IfModule !mod_rewrite.c>
    <IfModule mod_alias.c>
        RedirectMatch 302 ^/$ /index.php/
    </IfModule>
</IfModule>
```

Luciano



# Servicio de Mail
Se creó un servicio MailService que se instanció en AppController por lo tanto todos los controllers tienen acceso sin problema
Desde cualquier Controller se puede usar de la siguiente forma

**Uso Básico**

    $variables = array();
    $destinatario = 'info@gmail.com';
    $codigo = 'plantilla_registro_usuario'; 
    $this->mailing->enviarByCodigo($codigo,$destinatario,$variables);

**Codigos existentes hoy**:
 - plantilla_registro_usuario /* Sólo este esta implementado al 100% */
 - plantilla_compra_realizada /* Este se puede implementar */
 - plantilla_tracking_de_pedido /* Ni idea como funcionaría esto */
 - plantilla_compra_cancelada /* Ni idea como funcionaría esto */
 - plantilla_mensaje_del_sistema /* Ni idea como funcionaría esto */
 - plantilla_mensaje_del_consumidor /* Ni idea como funcionaría esto */

**Como variante sin usar templates**:

    $variables = array();
    $destinatario = 'info@gmail.com';
    $plantilla = '<p>Hola Juancito!!!</p>'; 
    $asunto = 'Soy un asunto';
    $this->mailing->enviar($plantilla,$destinatario,$asunto);


# Crontab
Se deben incluir en cada servidor los siguientes comandos para que corran en los tiempos establecidos:

**Comandos**:
 - php bin/console app:notificar-Stock
 - php bin/console app:abandonar-Carritos
 - php bin/console app:sync-Contabilium
 - php bin/console app:sync-VirtualSeller
 - php bin/console app:limpiar-papelera

Recordar que se deben adecuar a la configuración del servidor.

**Ejemplo**:
    php /home/signosecommerce/public_html/bin/console app:notificar-Stock >/dev/null 2>&1