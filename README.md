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