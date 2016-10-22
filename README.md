
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

# Configuración del router laravel Dingo Api

A continuación encontrará la plantilla del archivo de rutas laravel __(app/Http/routes.php)__


primero debemos obtener una instancia del router API para crear nuestros criterios de valoración.

    $api = app('Dingo\Api\Routing\Router');

Ahora tenemos que definir un grupo de versiones. Esto nos permite crear el mismo punto final para varias versiones deberían tenemos que cambiar las cosas por la vía.

    $api->version('v1', function ($api) {

    });

También puede tratar a este grupo como un grupo estándar de su marco determinado por el paso de una serie de atributos como el segundo parámetro.

     // API
     $api->group(['namespace'=>'App\Http\Controllers\Api'],function($api){

     }
Metodos protegidos por la autenticación de usuarios

        // Protected methods (require auth)
        $api->group(['middleware'=>'api.auth'],function($api){

        });

Metodos que nos permite hacer uso de peticiones externas CORS

            header('Access-Control-Allow-Origin: http://localhost:4200');
            header('Access-Control-Allow-Headers: Origin, Content-Type, Authorization');
            header('Access-Control-Allow-Methods: POST, GET, OPTIONS, PUT, PATCH, DELETE');


### Rutas final

        // API Routes come first
        $api = app('Dingo\Api\Routing\Router');
        $api->version('v1',function($api){

            header('Access-Control-Allow-Origin: http://localhost:4200');
            header('Access-Control-Allow-Headers: Origin, Content-Type, Authorization');
            header('Access-Control-Allow-Methods: POST, GET, OPTIONS, PUT, PATCH, DELETE');

            // API
            $api->group(['namespace'=>'App\Http\Controllers\Api'],function($api){
                // Auth
                $api->post('auth/login','Auth\AuthController@Login');
                $api->post('auth/token-refresh','Auth\AuthController@refreshToken');
                $api->post('users','Auth\UsersController@store');

                // Protected methods (require auth)
                $api->group(['middleware'=>'api.auth'],function($api){

                });

                // Public methods

            });
        });

        // Catchall - Displays Ember app
        Route::any('{catchall}',function(){
            return view('index');
        })->where('catchall', '(.*)');

### Configuración de Autenticación con JWT

Primero tenemos que crear el Controller de Autenticacion

        php artisan make:controller AuthController

dentro de la ruta __(app/Http/Controllers/)__ creamos la carpeta __Api__ que es donde va estar nuestro controler, como vamos a manejar versiones dentro de la carpeta __Api__ creamos otra carpeta llamada __V1__ con esto indicamos que esta es la version 1 de nuestro REST APi, ahora dentro de esta carpeta vamos a manejar todos nuestros Controller, pero creamos otra carpeta llamada __Auth__ que la que nos va manejar el uso de Autenticación de usuarios por medio del token y recargar el token

        ## __(app/Http/Controllers/)__
        mkdir Api
        cd Api
        mkdir V1
        cd V1
        mkdir Auth

ahora copiamos nuestro controller __AuthController__ anterior mente creado y lo copiamos en __(app/Http/Controllers/Api/V1/Auth/)__


Primero le cambiamos el namescape

    namespace app\Http\Controllers\Api\V1\Auth;

Le agregamos el uso de __Dingo Api__ y __JWT__

        use Dingo\Api\Http\Request;
        use Dingo\Api\Routing\Helpers;
        use Symfony\Component\HttpKernel\Exception\HttpException;
        use Symfony\Component\HttpKernel\Exception\UnauthorizedHttpException;
        use Tymon\JWTAuth\Exceptions\JWTException;
        use Tymon\JWTAuth\Exceptions\TokenInvalidException;

        ## tambien agregamos el uso de Helpers dentro de la Clase principal

         use Helpers;

Vamos a crear un metodo llamado __Login__ el cual se encargara del control de inicio de sessión y generara el Token

     public function Login(Request $request)
        {
            // grab credentials from the request
            $credentials = $request->only('email', 'password');

            try {
                // attempt to verify the credentials and create a token for the user
                if (! $token = \JWTAuth::attempt($credentials)) {
                    throw new UnauthorizedHttpException("Email address / password do not match");
                }
            } catch (JWTException $e) {
                // something went wrong whilst attempting to encode the token
                throw new HttpException("Unable to login");
            }

            // all good so return the token
            return $this->response->array(compact('token'));
        }
Y creamo otro metodo __refreshToken__ que es el que se encarga de atualizar el token

    public function refreshToken(Request $request)
        {
            $token  =   $request->get('token');
            if(!$token)
            {
                return $this->response->errorBadRequest('Token not provided');
            }
            try {
                $token  =   \JWTAuth::refresh($token);
            }
            catch(TokenInvalidException $e) {
                return $this->response->errorForbidden('Invalid token provided');
            }
            return $this->response->array(compact('token'));
        }
