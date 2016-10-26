<h3 align="center"><img src="resources/images/favicon.png" style="width=40px"></h3>
<h1 align="center">Slim Micro Framework</h1>
<h2 align="center">Manual para hacer CRUD</h2>

### Contenido de este manual
1. [Prerrequisitos](#prerrequisitos-para-usar-slim)<br>
2. [Instalación](#instalación)<br>
3. [Creación de Base de datos](#creación-de-base-de-datos)<br>
3.1. [Método en consola](#método-en-consola)<br>
3.2. [Método usando phpmyAdmin](#método-con-phpmyadmin)<br>
4. [Configuración de Slim](#configuración-de-slim)<br>
5. [Modelo](#modelo)<br>
6. [Controlador](#controlador)<br>
7. [Referencias](#referencias)<br>


### Prerrequisitos para usar Slim

- PHP 5.5 o posterior
- Un servidor web con reescritura de URLs 
- Sistema Manejador de Bases de Datos MySQL/MariaDB<sup>[1](#foot1)</sup>
- Eloquent (ORM)
- Respect Validation
- Twig (vistas)



### Instalación

La manera para instalar Slim recomendada por sus desarrolladores es mediante PHP Composer.

#### Instalación de Composer 

Para instalar Composer escribimos en consola el siguiente comando:

```
	$ curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer
```


o si se prefiere, se puede usar el siguiente script<sup>[2](#foot2)</sup>:

```sh
#!/bin/sh

EXPECTED_SIGNATURE=$(wget https://composer.github.io/installer.sig -O - -q)
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
ACTUAL_SIGNATURE=$(php -r "echo hash_file('SHA384', 'composer-setup.php');")

if [ "$EXPECTED_SIGNATURE" = "$ACTUAL_SIGNATURE" ]
then
    php composer-setup.php --quiet
    RESULT=$?
    rm composer-setup.php
    exit $RESULT
else
    >&2 echo 'ERROR: Invalid installer signature'
    rm composer-setup.php
    exit 1
fi

```
Guardado como `install-composer.sh` para ejecutarlo en terminal con el comando 

```
	$	sh install-composer.sh
```

**Nota**: Mediante este método, podemos mantener actualizado Composer, pero debes mover el archivo composer.phar a la carpeta `/usr/local/bin/` con el comando 

```
	# 	mv composer.phar /usr/local/bin/composer
```
de este modo podrás ejecutar Composer escribiendo solo `composer` en consola en vez de `php <Directorio>composer.phar`.


#### Instalación de Slim

Una buena ventaja de Slim es que proporciona un esqueleto básico sobre el que puedes comenzar a escribir tu aplicación, solo tienes que escribir en consola lo siguiente: 

```
	$ composer create-proyect slim/slim-skeleton crud-slim
```

Esto creará un nuevo directorio `crud-slim`con los archivos necesarios para comenzar a escribir la aplicación.

**Estructura del directorio** 

```
crud-slim
├── composer.json
├── composer.lock
├── CONTRIBUTING.md
├── dirstruct.txt
├── logs
│   ├── app.log
│   └── README.md
├── phpunit.xml
├── public
│   └── index.php
├── README.md
├── src
│   ├── dependencies.php
│   ├── middleware.php
│   ├── routes.php
│   └── settings.php
├── templates
│   └── index.phtml
├── tests
│   └── Functional
│       ├── BaseTestCase.php
│       └── HomepageTest.php
└── vendor/...

```

**Nota:** el directorio `vendor/` contiene muchos subdirectorios pero no es recomendado editar ninguno de los archivos que se contienen aquí ya que aquí está todo lo que composer maneja por nosotros y editarlos causaría daños en el proyecto.

Si ejecutamos `composer start`en el directorio de nuestra aplicación y abrimos nuestro navegador en la dirección `localhost:8080` aparecerá la siguiente vista
<img alt="vista inicial del esqueleto de Slim" src="resources/images/esqueleto.png">


### Creación de Base de datos


#### Método en Consola
Creamos una base de datos con el nombre `slim` 

```
$ mysql -u[nombre-de-usuario] -p
> CREATE DATABASE slim COLLATE = 'utf8_unicode_ci';
> \u slim
```

Agregamos la tabla `usuarios`.

```sql
>  CREATE TABLE usuarios (`id` BIGINT NOT NULL AUTO_INCREMENT,
	                     `nombre` VARCHAR (250) NOT NULL,
						 `correo` VARCHAR (250) NOT NULL,
						 `clave_acceso` VARCHAR (250) NOT NULL,
						 PRIMARY KEY (`id`));
```

#### Método con phpMyAdmin

Creamos la base de datos que usaremos para el crud:
	<img src="resources/images/1.png" alt="Creación de base de datos">

Creamos la tabla de usuarios:
	<img src="resources/images/2.png" alt="Creación de tabla usuarios">



### Configuración de Slim

Ahora que tenemos la base de datos, hay que agregarla a la configuración de Slim. Para esto, abrimos el archivo `settings.php` que  se encuentra en el directorio `src` y que contiene lo siguiente:

```php
<?php
return [
    'settings' => [
        'displayErrorDetails' => true, // set to false in production
        'addContentLengthHeader' => false, // Allow the web server to send the content-length header

        // Renderer settings
        'renderer' => [
            'template_path' => __DIR__ . '/../templates/',
        ],

        // Monolog settings
        'logger' => [
            'name' => 'slim-app',
            'path' => __DIR__ . '/../logs/app.log',
            'level' => \Monolog\Logger::DEBUG,
        ],
    ],
];


``` 
agregamos después de  `logger` la configuración de nuestra base de datos

```php

	//Configuración de base de datos para Slim
	'db' => [
		'host' => 'localhost',
		'user' => '<tu nombre de usuario en mysql>',
		'password' => '<tu contraseña>',
		'dbname' => 'slim'	
		'charset'   => 'utf8',
	    'collation' => 'utf8_unicode_ci',
        'prefix'    => '',
		],
		
```

### Modelo

Ahora hay que crear el _Modelo_ para la aplicación. Aunque Slim no sigue el patrón de diseño MVC (Modelo-Vista-Controlador) de un modo convencional, nos conviene tener un directorio exclusivo para cada componente, así que crearemos un directorio para el modelo dentro de `src/`con nuestro explorador de archivos o con el comando `mkdir models` desde el directorio `src`.

Como sabemos, Slim no cuenta con una herramienta para el Mapeo Objeto-Relacional por defecto. Sin embargo, nos permite agregar una de otro framework en PHP, por lo que usaremos _Eloquent_<sup>[3](#foot3)</sup> de Laravel.

Para agregar _Eloquent_ a nuestro CRUD primero debemos pedirle a composer que lo instale.

```sh
	$ composer require illuminate/database "~5.1"
```

Luego agregamos _Eloquent_ a las dependencias de la aplicación. Abrimos el archivo `dependencies.php` que está en el directorio `src`y le agregamos 

```php
$container['db'] = function ($container) {
    $capsule = new \Illuminate\Database\Capsule\Manager;
    $capsule->addConnection($container['settings']['db']);

    $capsule->setAsGlobal();
    $capsule->bootEloquent();

    return $capsule;
};
```

Creamos la clase _Usuario_ dentro del directorio models.

 ```php
 <?php
namespace Models;

//importa Eloquent para usarlo en el modelo
use Illuminate\Database\Eloquent\Model as Eloquent;

class Usuario extends Eloquent
{
    // Define la llave primaria de la tabla usuarios
    protected $primaryKey = 'id';

    // Define el nombre de la tabla 
    protected $table = 'usuarios';
}
```


### Controlador

También crearemos un directorio `controllers` dentro de `src`para los controladores de la aplicación. Una vez creado el directorio, crearemos dentro de él una clase de _controlador_ genérica para usarla como base del resto de los controladores.

```php
<?php

namespace Controllers;

use \Slim\Container;

/**
 * Clase de controlador genérico que sirve como base para la aplicación
 */

class Controller
{

    // la aplicación
    protected $app;
    
    // el contenedor de inyección de dependencias de la aplicación
    protected $container;

    /**
     * Constructor de la clase Controller
     * @param \Slim\Container - DIC
     */
    public function __construct(Container $container)
    {
        $this->app = Slim\Slim::getInstance();
        $this->container = $container;
    }

    
    /**
     * Toma la variable del método GET de una petición http
     * @param type string $key - el parametro que se busca
     */
    public function httpGet($key)
    {
        // busca el parametro de la consulta en la consulta completa
        if(isset($this->container->request->getQueryParams()[$key])) {
            return $this->container->request->getQueryParams()[$key];
        } else {
            return null; // si no hay parametro regresa un objeto nulo
        }
    }

    /**
     * Toma la variable del método POST de una petición http
     *  @param type string $key - el parametro que se busca
     */
    public function httpPost($key)
    {
        // busca el parametro en el cuerpo parseado de la petición
        if (isset($this->container->request->getParsedBody()[$key])) {
            return $this->container->request->getParsedBody()[$key];
        } else {
            return null;
        }
    }

    /**
     * Convierte un objecto en una cadena en formato JSON
     * @param object $data - el objeto a convertir
     */
    public static function($data)
    {
        // si el parametro que se recibe no es objeto, regresa null
        if (!is_object($data)) {
            return null;
        } else {
            /* codifica el objeto a JSON de manera legible
               y con la constante de final de linea para manejar los saltos de linea en multiplataforma*/
            return json_encode($data, JSON_PRETTY_PRINT) . PHP_EOL;
        }
    }
	
   /**
     * Crea una cadena de error en formato de JSON
     * @param type string $code - código del error
     * @param type string $path - ruta del error
     * @param type string $status - estado del error
     * @param type string $extra - información adicional
     * @return string - el error en formato JSON
     */
    public static function error($code, $path, $status, $extra = '')
    {
        $error = new \StdClass;
        
        $error->error = [
            'code' => $code,
            'path' => $path,
            'status' => $status
        ];
        
        if ($extra) {
            $error->error['extra'] = $extra;
        }
        return self::encode($error);
    }
			
    /**
     * Evalua una lista de validaciones
     * @param $valid - lista de validaciones
     */
    public static function valida($valid)
    {
        if (is_array($valid){
            foreach($valid as $v){
                if ($v == false){
                    return false; // regresa false si una validación es falsa
                } 
            } 
            return true;
        } else { return false; } // si no es un arreglo
    }
}
```

Como nuestros controladores harán validaciones, instalamos la herramienta para validaciones Respect Validation mediante composer.

```
	$ composer require respect/validation
```

Ahora creamos el controlador para el usuario, también el directorio `controllers`.

```php
<?php

use Models; // para usar el modelo de usuario
use Respect\Validation\Validator as valida; // para usar el validador de Respect

class Usuario extends Controller
{

    /**
     * Obtiene los atributos para el método http POST 
     * para crear/actualizar un usuario
     *
     */
    public function getAtr()
    {
        $atr = [
            'id' => $this->httpPost('id'),
            'nombre' => $this->httpPost('nombre'),
            'correo' => $this->httpPost('correo'),
            'clave_acceso' => $this->httpPost('clave_acceso')
        ];
        return $atr;
    }

    /**
     * Hace las validaciones para un POST
     * @param $atr - los atributos a evaluar
     */
    public function validaPOST($atr)
    {
        $valid = [
            // verifica que la id sea un entero
            valida::intVal()->validate($atr['id']),
            // verifica que se reciba una cadena de al menos longitud 2
            valida::stringType()->length(2)->validate($atr['nombre']),
            // verifica que se reciba un correo
            valida::email()->validate($atr['correo']),
            // verifica que no esté en blanco la contraseña
            valida::notBlank()->validate($atr['clave_acceso'])
        ];
                                                 
    }

    /**
     * Hace las validaciones para un método PATCH
     * @param type array $atr - atributos a evaluar
     */
    public function validaPATCH($atr)
    {
        // es análogo al de POST
        $valid = [
            valida::intVal()->validate($atr['id']),
            valida::stringType()->length(2)->validate($atr['nombre']),
            valida::email()->validate($atr['correo']),
            valida::notBlank()->validate($atr['clave_acceso'])
        ];
    }

    /**
     * Obtiene todos los usuarios de la tabla usuarios
     */
    public function todos()
    {
        $usuarios = Models\Usuario::get(); // usa la función get() de Eloquent

        if($usuarios) {
            echo self::encode($usuarios); // muestra los usuarios agregados
        }    
    }

    /**
     * Obtiene un usuario por su id
     * @param type Slim\Http\Request $request - la solicitud http
     * @param type Slim\Http\Response $response - la respuesta http
     * @param type array $args - argumentos para la función
     */
    public function getID($request, $response, $args)
    {
        $id = $args['id'];

        $valid = [v::intVal()->validate($id)]; //hace la validación de la id

        // si la validación resulta cierta
        if ($this->validate($valid) == true){
            $usuario = Models\Usuario::find($id); // busca la id en la tabla
            if ($usuario){
                // si encuentra al usuario lo muestra
                echo self::encode($usuario);
            } else {
                // si no hay un usuario regresa un error 404 (not found)
                $status = 404; 
                echo->$this->error('get#usuario{id}',
                                   $request->getUri()->getPath(),
                                   $status);
                return $response->withStatus($status); 
            }
        } else {
            // si la validación es falsa, regresa un error de bad request 
            return $response->withStatus(400);
        }
    }   
}

```
#### Crear
Recordemos que la tabla usuario tiene como atributos una _id_, un _nombre_,un _correo_ y una _contraseña_. Como `id` es un atributo auto incrementable, solo necesitamos ingresar los otros 3 campos a la base de datos y lo haremos con esta función:

```php

```


### Referencias

> <a name="foot1">1</a>: Mariadbcom. (2016). Mariadbcom. Retrieved 25 September, 2016, from https://mariadb.com/blog/why-should-you-migrate-mysql-mariadb. <br>
> <a name="foot2">2</a>: How to install Composer programmatically?#. (n.d.). Retrieved September 25, 2016, from https://getcomposer.org/doc/faqs/how-to-install-composer-programmatically.md. <br>
> <a name="foot3">3</a>:Eloquent: Getting Started - Laravel - The PHP Framework For Web Artisans. (n.d.). Retrieved September 29, 2016, from https://laravel.com/docs/5.1/eloquent <br>

