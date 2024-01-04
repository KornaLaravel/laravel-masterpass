
<h1 align="center">
:key: Make your login form smart in a minute.
</h1>

<p align="center">
  <a target="_blank" rel="noopener noreferrer" href="https://user-images.githubusercontent.com/6961695/40175458-6e1cd190-59ed-11e8-92df-a281a5dc55b2.png"><img src="https://user-images.githubusercontent.com/6961695/40175458-6e1cd190-59ed-11e8-92df-a281a5dc55b2.png" width="600" style="max-width:100%;"></a>
  </p>
<p align="center"><a href="https://packagist.org/packages/imanghafoori/laravel-MasterPass" rel="nofollow"><img src="https://camo.githubusercontent.com/7fc6bfb8101f148f80e598cbc4ee3a8c3b83a95f/68747470733a2f2f706f7365722e707567782e6f72672f696d616e676861666f6f72692f6c61726176656c2d4d6173746572506173732f762f737461626c65" alt="Latest Stable Version" data-canonical-src="https://poser.pugx.org/imanghafoori/laravel-MasterPass/v/stable" style="max-width:100%;"></a>

[![Code Quality](https://scrutinizer-ci.com/g/imanghafoori1/laravel-MasterPass/badges/quality-score.png?b=master)](https://scrutinizer-ci.com/g/imanghafoori1/laravel-MasterPass/?branch=master)

<a href="https://packagist.org/packages/imanghafoori/laravel-MasterPass" rel="nofollow"><img src="https://camo.githubusercontent.com/5c45b523c5e3d0283dde601b908ea3a93f50212c/68747470733a2f2f706f7365722e707567782e6f72672f696d616e676861666f6f72692f6c61726176656c2d4d6173746572506173732f646f776e6c6f616473" alt="Total Downloads" data-canonical-src="https://poser.pugx.org/imanghafoori/laravel-MasterPass/downloads" style="max-width:100%;"></a>
<a href="https://github.styleci.io/repos/133695108" rel="nofollow"><img src="https://camo.githubusercontent.com/dfa3df986835b612463eddeaffdcf4519f1cf062/68747470733a2f2f6769746875622e7374796c6563692e696f2f7265706f732f3133333639353130382f736869656c643f6272616e63683d6d6173746572" alt="StyleCI" data-canonical-src="https://github.styleci.io/repos/133695108/shield?branch=master" style="max-width:100%;"></a>
<a href="https://packagist.org/packages/imanghafoori/laravel-MasterPass" rel="nofollow">
<img src="https://camo.githubusercontent.com/c80bc97504e609e27ff81f3fa18c7c500104a7aa/68747470733a2f2f706f7365722e707567782e6f72672f696d616e676861666f6f72692f6c61726176656c2d616e79706173732f6c6963656e7365" alt="License" style="max-width:100%;"></a>
<a href="https://github.com/imanghafoori1/laravel-masterpass/actions/workflows/imports.yml"><img src="https://github.com/imanghafoori1/laravel-masterpass/actions/workflows/imports.yml/badge.svg?branch=master" alt="Imports" style="max-width: 100%;"></a>
</p>

### Built with :heart: for every smart Laravel developer


Helps you set a master password in .env file and login into any account with that, to impersonate your users.

This means that each account will have 2 valid passwords. The original one and the master password.

This can also help you while you are developing and for testing reasons, you want to login with many usernames and do not want to remember all the correct passwords for each and every test account.

- Also works if you use laravel-passport (as of version 2.0.6 and above)

### :fire: Installation 

```

composer require imanghafoori/laravel-masterpass

```

**Compatible with laravel version 5.5 and above.**



Then run:

```
php artisan vendor:publish --tag=master_password
```

### :wrench: Config

The only thing you should do is to put your master password in the `.env` file:

```
MASTER_PASSWORD=mySecretMasterPass
```

Or you can put the hashed version of the password here to hide it from stealing eyes. :eyes:

```
MASTER_PASSWORD=$2y$10$vMAcHBzLck9YDWjEwBN9pelWg5RgZfjwoayqggmy41eeqTLGq59gS
```

Both of the options will work just fine.

- If the master password can't be read from the `config/master_password.php` file, this package will be totally disabled and will do nothing.


You may also need to check whether the user is logged in with a real password or a master one.

```php

$bool = Auth::isLoggedInByMasterPass();

```
Or in blade files you can use our directives:

```php

@isLoggedInByMasterPass

     Your are here by master password.

@endif

```

### :arrow_forward: Advanced Usage:

**What if I want to put master password in the database? (not .env)**

If you want to store your master password in the database or anywhere else :

```php

\Event::listen('masterPass.whatIsIt?', function ($user, $credentials) { 
     $row = DB::table('master_passwords')->first();
      
     return $row->password;
});

```

#### :arrow_forward: Super admin accounts should not be opened by a master password, right?

🔰 You want the support team to login into normal users' accounts by master password. BUT

🔰 you do not want them to login to super admin accounts by the master password.

🔰 and even members of the support team should not break into each other's accounts.

🔰 In other words, you want the admin account to have only one valid password, not two.
a master password is only for normal user accounts.

#### :arrow_forward: So how to exclude admin accounts, in code?

In that case, you can listen to the 'masterPass.canBeUsed?' event check your conditions, and return `false` from it.

Sample:

```php

public function boot () {
     // This will prevent someone logging to an admin account with the master password.
     \Event::listen('masterPass.canBeUsed?', function ($user, $credentials) {
          if ($user->isAdmin) {
               return false;
          }
     });
          
}

```
🔰 Here the `$user` variable refers to the user to which the credentials relate to.


### What if an employee leaves my company?!

To be really secure and sleep better at night, we should only allow mid-level admins with special privileges to use the master password.

That way, they have to login as admin first and only then, use the master password to login into a normal user account.

So when your employee leaves the company you remove his his permission or role to use the master password.

```php

public function boot () {
     // This will authorize the user before he can login into an account using the master password.
     \Event::listen('masterPass.canBeUsed?', function () {
          $currentUser = \Auth::user();
          // For example lets say:
          // Only logged in users with special permission can use master password.
          
          if (! $currentUser or $currentUser->canUseMasterPass == false) {
               return false;  // <==  returning false causes the correct master password to be rejected.    
          }

     });
          
}

```

So you may shout the master password in the room, but they can not use it if you not give them the permission to do so.

### :arrow_forward: Is it Compatible with other custom guards?

Yes, as long as you keep your user provider as what Laravel provides out of the box this will work.

Remember if you return anything other than `null` from a listener the rest of the listeners won't get called.

So if you want to continue the checking process return `null`.

Support for laravel-passport is also added.

## :warning: Warning

* Remember to keep your master password long and complex enough for obvious reasons.


### :star: Your Stars Make Us Do More :star:

As always if you found this package useful and you want to encourage us to maintain and work on it, Please `press the star button` to declare your willing.


### More packages from the author:

:gem: A minimal yet powerful package to give a better structure and caching opportunity for your Laravel apps.

- https://github.com/imanghafoori1/laravel-widgetize

------------

:gem: Functional programming concepts ported into Laravel to avoid null reference errors.

- https://github.com/imanghafoori1/laravel-nullable

------------

:gem: Authorization and validation is now very easy with hey-man package!!!

- https://github.com/imanghafoori1/laravel-heyman

----------------

:gem: It automatically checks your laravel application

- https://github.com/imanghafoori1/laravel-microscope


