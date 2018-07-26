# Laravel Hey Man

## A package to help you write expressive, defensive code in a functional manner

![image](https://user-images.githubusercontent.com/6961695/43092242-c26b141a-8ec1-11e8-8108-4e0cc63d0522.png)

## And it works !!!

<a href="https://scrutinizer-ci.com/g/imanghafoori1/laravel-heyman"><img src="https://img.shields.io/scrutinizer/g/imanghafoori1/laravel-heyman.svg?style=round-square" alt="Quality Score"></img></a>
[![code coverage](https://codecov.io/gh/imanghafoori1/laravel-heyman/branch/master/graph/badge.svg)](https://codecov.io/gh/imanghafoori1/laravel-heyman)
[![Maintainability](https://api.codeclimate.com/v1/badges/9d6be7b057103cb14410/maintainability)](https://codeclimate.com/github/imanghafoori1/laravel-heyman/maintainability)
[![Build Status](https://travis-ci.org/imanghafoori1/laravel-heyman.svg?branch=master)](https://travis-ci.org/imanghafoori1/laravel-heyman)
[![StyleCI](https://github.styleci.io/repos/139709518/shield?branch=master)](https://github.styleci.io/repos/139709518)
[![Latest Stable Version](https://poser.pugx.org/imanghafoori/laravel-heyman/v/stable)](https://packagist.org/packages/imanghafoori/laravel-heyman)
[![Software License](https://img.shields.io/badge/license-MIT-brightgreen.svg?style=round-square)](LICENSE.md)



## Installation

```

composer require imanghafoori/laravel-heyman

```



Imagine your boss comes to you and says :

### Hey man,
### When you go to login form, You should be guest,
### Otherwise you must get redirected to '/panel',
###  Write the code for me, just now

and you write code like this to implement what your boss wanted.


```php


HeyMan::whenYouMakeView('auth.login')->youShouldBeGuest()->otherwise()->redirect('/panel');


```


### That is what this package does for you + a lot more things...



So now we can use this gate to authorize and stop the user in various moments of the application life cycle, including:
- 1- Right after a Route is Matched
- 2- Right before a blade file is going to be rendered. For example by calling: `view('myViewFile');` 
- 3- Right before an eloquent model is going to be `read` or `created` or `updated` or `deleted`
- 4- When any custom event is fired by calling `event('myEvent');`


You can put these codes in `AuthServiceProvider.php` (or any other service provider) `boot` method to take effect:

## Watching Urls

```php
HeyMan::whenYouVisitUrl(['/welcome', '/home'])->...   // you can pass an Array
HeyMan::whenYouVisitUrl( '/welcome', '/home' )->...   // variable number of args
HeyMan::whenYouVisitUrl('/admin/articles/*')->...     // or match by wildcard
```


## Watching Route Names

```php
HeyMan::whenYouVisitRoute('welcome.name')->...
HeyMan::whenYouVisitRoute('welcome.*')->...                 // or match by wildcard
```


## Watching Controller Actions

```php
HeyMan::whenYouCallAction('HomeController@index')->...
HeyMan::whenYouCallAction('HomeController@*')->...          // or match by wildcard
```

## Watching Blade files
```php 
 HeyMan::whenYouMakeView('article.editForm')->...     // also accepts an array
 HeyMan::whenYouMakeView('article.*')->...            // You can watch a group of views
 ```
 
 ## Watching Custom Events
```php
HeyMan::whenEventHappens('myEvent')->...
```

## Watching Eloquent Model Events
```php
HeyMan::whenYouSave(\App\User::class)->...
HeyMan::whenYouFetch(\App\User::class)->...
HeyMan::whenYouCreate(\App\User::class)->...
HeyMan::whenYouUpdate(\App\User::class)->...
HeyMan::whenYouDelete(\App\User::class)->...
```
 
*In case the gate returns `false` an `AuthorizationException` will be thrown.
*(If it is not the thing you want, do not worry you can customize the action very easily, we will discuss shortly.)


This way gate is checked after `event('myEvent')` is executed any where in our app





## What should be checked:

### 1 - Gates

```php
HeyMan::whenYouVisitUrl('/home')->thisGateShouldAllow('hasRole', 'param1')->otherwise()->...;
HeyMan::whenYouVisitUrl('/home')->thisGateShouldAllow('SomeClass@someMethod', 'param1')->otherwise()->...;
```

Passing a Closure as a Gate:

```php
$gate = function($user, $role){
    /// some logic
    return true;
}

HeyMan::whenYouVisitUrl('/home')->thisGateShouldAllow($gate, 'editor')->otherwise()->...;
```

### 2 - Authentication:
```php
HeyMan::whenYouVisitUrl('/home')->youShouldBeGuest()->otherwise()->...;
HeyMan::whenYouVisitUrl('/home')->youShouldBeLoggedIn()->otherwise()->...;
```

### 3 - Checking A Closure:
```php
$callback = function($params){
    /// some logic
    return true;
}
HeyMan::whenYouVisitUrl('home')->thisClosureMustPass($callback, ['param1'])->otherwise()->...;
```


--------------------


## Reactions:

### 1 - Deny Access
```php
HeyMan::whenSaving(\App\User::class)->thisGateShouldAllow('hasRole', 'editor')->otherwise()->weDenyAccess();
```
By calling `weDenyAccess` method an `AuthorizationException` will be thrown if needed


### 2 - Redirect
```php
HeyMan::whenYouVisitUrl('/login')-> ... ->otherwise()->redirect()->to(...);
HeyMan::whenYouVisitUrl('/login')-> ... ->otherwise()->redirect()->route(...);
HeyMan::whenYouVisitUrl('/login')-> ... ->otherwise()->redirect()->action(...);
HeyMan::whenYouVisitUrl('/login')-> ... ->otherwise()->redirect()->intended(...);
HeyMan::whenYouVisitUrl('/login')-> ... ->otherwise()->redirect()->guest(...);
```

### 3- Throw Exception:
```php

$exceptionType = AuthorizationException::class;
$msg = 'My Message';

HeyMan::whenYouVisitUrl('/login')->youShouldBeGuest()->otherwise()->throwNew($exceptionType, $msg);
```

### 4- Abort:
```php
HeyMan::whenYouVisitUrl('/login')-> ... ->otherwise()->abort(...);
```

### 5- Send Response:

Calling these functions generate exact same response as calling them on the `response()` helper function like this:
 
`return response()->json(...);`

```php
HeyMan::whenYouVisitUrl('/login')-> ... ->otherwise()->response()->json(...);
HeyMan::whenYouVisitUrl('/login')-> ... ->otherwise()->response()->view(...);
HeyMan::whenYouVisitUrl('/login')-> ... ->otherwise()->response()->jsonp(...);
HeyMan::whenYouVisitUrl('/login')-> ... ->otherwise()->response()->make(...);
HeyMan::whenYouVisitUrl('/login')-> ... ->otherwise()->response()->download(...);
```
