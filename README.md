# Creción de Proyectos Laravel con:

+ [Laravel 5.2 (LTS)](https://laravel.com/docs/5.2)
+ [Dingo API 1.0.0 (beta)](https://github.com/dingo/api)
+ [Tymon jwt-auth 0.5.9](https://github.com/tymondesigns/jwt-auth)
+ PHP 5.5.9+


# Iniciando 

## Instalamos laravel

Creamos un nuevo proyecto con laravel

    composer create-project --prefer-dist laravel/laravel blog "5.2.*"
    
Añadir su configuración de base de datos para el archivo __.env__ mediante la actualización de las siguientes líneas:
    
    DB_HOST=localhost
    DB_DATABASE=homestead
    DB_USERNAME=homestead
    DB_PASSWORD=secret

## Instalamos Dingo API

El paquete __Dingo API__ tiene una amplia [documentación disponible](https://github.com/dingo/api/wiki), se utiliza para la creacion de REST APi 

    composer require dingo/api:1.0.x@dev
    
Después de tener instalado API  Dingo hay que agregarlo a la lista de proveedore de laravel __(config/app.php)__

    Dingo\Api\Provider\LaravelServiceProvider::class,
    

Publique el archivo de configuración __Dingo API__
    
    php artisan vendor:publish --provider="Dingo\Api\Provider\LaravelServiceProvider"
    
Añadir algunas opciones básicas de configuración en el fichero de .env

    API_PREFIX=api
    API_STANDARDS_TREE=vnd
    API_VERSION=v1
    API_DEBUG=true
    API_NAME="My API"
    API_DEBUG=true
    
## Instalamos Tymon JWT-Auth    

El paquete __Tymon JWT-Auth__ tiene una amplia [documentación disponible](https://github.com/tymondesigns/jwt-auth/wiki), se utiliza para la autenticación por medio Token

    composer require tymon/jwt-auth
    
Después de tener instalado __Tymon JWT-Auth__ hay que agregarlo a la lista de proveedore de laravel __(config/app.php)__

    Tymon\JWTAuth\Providers\JWTAuthServiceProvider::class,
    
Añadir __JWTAuth__ a la sección de alias  laravel __(config/app.php)__
  
    'JWTAuth' => 'Tymon\JWTAuth\Facades\JWTAuth',
 
Publique el archivo de configuración __JWTAuth__

    php artisan vendor:publish --provider="Tymon\JWTAuth\Providers\JWTAuthServiceProvider"

Generamos la clave secreta de __JWTAuth__

    php artisan jwt:generate


Ahora que la JWT autenticación y paquetes de API están instalados, tenemos que configurar la API para utilizar JWT por su método de autenticación. Afortunadamente, esto se puede hacer muy fácilmente en el archivo de configuración API se encuentra en __(config/api.php)__


    ## Buscamo la siguiente linea
    'auth' => [
   
    ],
    
    ### y agregamos esta linea 
    
    'jwt' => 'Dingo\Api\Auth\Provider\JWT',
    
    ## Nos quedaria así
    
    'auth' => [
       'jwt' => 'Dingo\Api\Auth\Provider\JWT',
    ],
    
 
 
## configuración y uso de la API JSON

JSON API es un estándar para las API que utilizan datos JSON y ha sido el adaptador de ajuste por defecto en Ember desde la versión 2.0. Con el fin de hacer que los datos de retorno de la API JSON API de formato sólo tenemos que configurar Dingo API para utilizar el JsonApiSerializer que viene con.

en __(app/Providers/AppServiceProvider.php)__

Primero agregue las siguientes instrucciones de uso:

    use Dingo\Api\Transformer\Adapter\Fractal;
    use Illuminate\Support\ServiceProvider;
    use League\Fractal\Manager;
    use League\Fractal\Serializer\JsonApiSerializer;
    
    ### Añadir lo siguiente al método de __boot()__ dentro de este proveedor de servicios:
    
    // API uses JSON API
    $this->app['Dingo\Api\Transformer\Factory']->setAdapter(function ($app) {
         $fractal = new Manager();
         $fractal->setSerializer(new JsonApiSerializer());
         return new Fractal($fractal);
    });
    
# configuración de Routas con Dingo Api

