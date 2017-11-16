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

Més elaborat: tot un sistema Oauth 