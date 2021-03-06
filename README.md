[![Build Status](https://travis-ci.org/Roemerb/php7-rest.svg?branch=master)](https://travis-ci.org/Roemerb/php7-rest)

# php7-rest

php7-rest is a very simple Http REST client with some optimization for PHP v7.0+. It's not necessarily intended to be very feature-rich or even to be widely used, but merely to provide a simple API for specifying REST API behaviour. 


### Install

If you'd like to install this library, I recommend using Composer:

```shell
composer require roemeb/php7-rest
```

Though, as this library does not have any dependencies - it only requires `ext-curl` -, you can also just clone this repository and start using `src/RestClient.php`.

### Usage

**Initialisation**

Initialize a client for an API like so:

```php
require_once('vendor/autoload.php');

use RestClient\RestClient;

$client = new RestClient(RestClient::HTTPS, 'api.myhost.com', ['version' => 2]);
```

You only have to specify the scheme and host. If you need to use any kind of authentication, you can do this by adding headers. Generating oauth tokens is not currently part of this library.

```shell
$token = Oauth::gen_token($credentials);

$client->addHeader('Authorization', 'Bearer ' . $token);
```

**Options**

There are some options that can be specified upon initialisation:

- `version` Version of your API. Will be appended to host as path (i.e. `myhost.com/v2/`)
- `port` Does your API run on an obscure port? Specify it here.
- `cert` File path to certificate bundle that will be loaded to avoid curl error 60
- `content_type` Custom value for the `Content-Type` header. Defaults to `application/json`. Set to `false` if you don't want this header at all.
- `user_agent` Custom value for the `User-Agent` header. Defaults to `Php7Rest/{Version}`. Set to `false` if you don't want this header at all.
- `insecure` Set to true to dissable curl host verification.

The header options may also be set by using `$client->addHeader($name, $value)`. It will override the defaults.

**Resources**

Querying data can be done in one of two ways. Either you can register resources which will be magically called, or your existing resource class can extend the `RestResource` class provided in this library.

**Registering resources**

Registering resources is quite simple. Simply provide a name and an array of methods this resource supports. Additionally, if your resource supports non-default methods, you can register these as well. If these require an ID, it will be included in the request automatically.

```php
$client->register('people', ['list', 'get', 'create']);
$client->registerMethod('people', [
	'name' 		=> 'address',
	'method' 		=> '{id}/formatted_address',
	'http_method'	=> RestClient::HTTP_GET // Or just 'GET'/'get'/'Get'
]);

/**
* This will call:
* GET https://api.myhost.com/people/v2/{id}
*/
$client->people->get($id);

// We can call custom methods the same way
$client->people->address($id);
```

**Extending `RestResource`**

```php
class Person extends RestResource
{
	public function __construct()
	{
		// Just give the parent constructor some information
		parent::__construct($restClient, strtolower(__CLASS__);
	}
}

$people = new Person();
$people->get($id);
```

**Closures**

All methods support closures, so you don't have to wait for a response from an API to continue your business logic. The response will be included as an argument to your function. Use closures as follows:

```php
// Initiate an API call...
$client->people->get($id, function($response) {
	// Do something with $response
});

// ...and continue to do other stuff in the meantime.
```

