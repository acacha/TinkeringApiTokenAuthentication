HTTP REQUEST HEADERS
--------------------

CSRF tokens:
- X-CSRF-TOKEN: Plain value. Meta field on HTML
- X-XSRF-TOKEN: Encrypted value . Laravel utiltiza una Encrypted cookie amb el mateix nom.
Examples:
- X-CSRF-TOKEN:T65UPXYzRHZgje7qxXuXoXuRAiiwwVjV5hVB6Dzi
- X-XSRF-TOKEN:eyJpdiI6IjVhVDg2YjBmQ3RrSHRlTHArVHZcL0N3PT0iLCJ2YWx1ZSI6IlAxbHFIa2RyaXlocHMyWlM1c1pKODNxUXVxYnE2NEk4ZExGbnR0R2t6QUdDUTBacXZmVHNleEE3K2Q0OVRLaVYxWGo1RnFjNjkxbUxGblwvb3VjQmFJUT09IiwibWFjIjoiZDlkYTM0ZjQ2NjY0OGVlYzZmNGE4YjNiMDMyMTUxNTExNjAyMGI1MWM5MTE2NDQ3ODQxN2VhYzQxYWQzMGY0YyJ9

```bash
php artisan tinker
encrypt('T65UPXYzRHZgje7qxXuXoXuRAiiwwVjV5hVB6Dzi')
"eyJpdiI6Imk3NTdKTGpVbUl2b2dxV25CS3diWVE9PSIsInZhbHVlIjoiaHA2alhmNExRUjFkRlRNQ0tCdjRRV3NoNGU3S0lmdnM0QWlcL2Z1ZEhuQlF5ZGpPOVh3OXFaUTdaMkhzUWhoSVdpUXpoTXVhTGpoazdweldKcCtXc2hRPT0iLCJtYWMiOiJlYzkxYzA2YjVlMWU0ZTg4NDdjZDM2ODAxODg1MzVhYzFkNDA4MTRhYmU2NzI3ODUyMmM0MGZhOTM4MDdhNDlhIn0="
```

Els tokens CSRF no són tokens autenticació!

Si comenteu la línia (middleware que encripta les cookies):

```
//            \App\Http\Middleware\EncryptCookies::class,
```

Del fitxer app/Http/Kernel.php els dos headers tindràn el mateix valor.

El middleware encarregat de posar la cookie i comprovar CSRF és:

```
//            \App\Http\Middleware\VerifyCsrfToken::class,
```

Si el treieu el únic camp a les peticions axios serà el no xifrat:

- X-CSRF-TOKEN:WqlYHAhYS9JkWw6fG9IGwVvfRHa7MO4ahwD8vbSm

X-XSRF-TOKEN no estarà


AXIOS
-----

Utilitza directament el valor de la cookie XSRF-TOKEN i el posa al header axios per defecte

// `xsrfCookieName` is the name of the cookie to use as a value for xsrf token
  xsrfCookieName: 'XSRF-TOKEN', // default

LARAVEL GUARDS
--------------

Dos guards i 3 drives:

- web: El login comú aplicat per auth a middleware grup web. Driver session
- API:
  - Driver token: token autenticació
  - Driver passport: Oauth server

Es pots canviar la configuració al fitxer config/auth.php

```
'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],

        'api' => [
            'driver' => 'token',
            'provider' => 'users',
        ],
    ],
```

La request ha de tenir (observeu importància HTTPS per evitar sniff amb atacs MITM):

En ordre de preferència
- Primer Query string 'api_token'
- 2 qualsevol input (POST object)
- Bearer Token, Header autorització amb:
  - Authorization: Bearer TOKEN HERE
  - Authorization: TOKEN HERE
- Password
  - HEADER PHP_AUTH_PW

Codi Laravel

```php
 Illuminate\Auth\TokenGuard
 
  /**
      * Get the token for the current request.
      *
      * @return string
      */
     public function getTokenForRequest()
     {
 
         $token = $this->request->query($this->inputKey);
 
         if (empty($token)) {
             $token = $this->request->input($this->inputKey);
         }
 
         if (empty($token)) {
             $token = $this->request->bearerToken();
         }
 
         if (empty($token)) {
             $token = $this->request->getPassword();
         }
 
         return $token;
     }
```

API TOKEN AUTHENTICATION
-------------------------

https://gistlog.co/JacobBennett/090369fbab0b31130b51

Bàsicament falta afegir un token a cada usuari (a la base de dades)

A la migració users afegir:

```php
	$table->string('api_token', 60)->unique();
```

Afegir el camp 'api_token' al fillable del model User. 

SEGURETAT: També afegir a hidden!!!!!!!!!!!

Modificar el fitxer *app/Http/Controllers/Auth/RegisterController.php* i afegir el camp api_token:

```php
// app/Http/Controllers/Auth/RegisterController.php
protected function create(array $data)
    {
        return User::create([
            'name' => $data['name'],
            'email' => $data['email'],
            'password' => bcrypt($data['password']),
            'api_token' => str_random(60)
        ]);
    }
```


LARAVEL PASSPORT
----------------

Més elaborat: tot un sistema Oauth. Només cal seguir les passes de:

https://laravel.com/docs/5.5/passport

Hi ha un middleware **\Laravel\Passport\Http\Middleware\CreateFreshApiToken::class**

Llegiu:

https://laravel.com/docs/5.5/passport#consuming-your-api-with-javascript

Crea una cookie **laravel_token** amb un JWT xifrat que s'utilitza per fer autenticació

LARAVEL PASSPORT GUARD
----------------------

Fitxer Laravel\Passport\Guards\TokenGuard

Mètode user():

```
 /**
     * Get the user for the incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  Request  $request
     * @return mixed
     */
    public function user(Request $request)
    {
        if ($request->bearerToken()) {
            return $this->authenticateViaBearerToken($request);
        } elseif ($request->cookie(Passport::cookie())) {
            return $this->authenticateViaCookie($request);
        }
    }
```

Obté de la cookie (posada pel middleware CreateFreshApiToken) o pel sistema de header authoritzacio

JWT
---

ES pot veure el format i pegar el token de Laravel aquí:

https://jwt.io/



Paquet: firebase/php-jwt

Laravel utilitza paquet: "firebase/php-jwt": "~3.0|~4.0|~5.0",

Docs:
- https://github.com/firebase/php-jwt
- https://jwt.io/

OAUTH
-----

## Another client. Pure Vue.js Javascript app (axios)

TODO

## Another client. PHP app with Guzzle

- https://scotch.io/@neo/getting-started-with-laravel-passport

Aplicació PHP

GRANT_TYPE: Password

```php
require "vendor/autoload.php";

$client = new GuzzleHttp\Client;

try {
    $response = $client->post('http://todos.dev/oauth/token', [
        'form_params' => [
            'client_id' => 2,
            // The secret generated when you ran: php artisan passport:install
            'client_secret' => 'fx5I3bspHpnuqfHFtvdQuppAzdXC7nJclMi2ESXj',
            'grant_type' => 'password',
            'username' => 'johndoe@scotch.io',
            'password' => 'secret',
            'scope' => '*',
        ]
    ]);

    // You'd typically save this payload in the session
    $auth = json_decode( (string) $response->getBody() );

    $response = $client->get('http://todos.dev/api/todos', [
        'headers' => [
            'Authorization' => 'Bearer '.$auth->access_token,
        ]
    ]);

    $todos = json_decode( (string) $response->getBody() );

    $todoList = "";
    foreach ($todos as $todo) {
        $todoList .= "<li>{$todo->task}".($todo->done ? '✅' : '')."</li>";
    }

    echo "<ul>{$todoList}</ul>";

} catch (GuzzleHttp\Exception\BadResponseException $e) {
    echo "Unable to retrieve access token.";
}
```

'''Resources'''
:*'''REsource'