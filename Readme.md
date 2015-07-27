# HTTP Client

#### A very lightweight PHP 5.4+ library for issuing HTTP requests, with a fluent API.

### Runtime requirements

-  PHP >= 5.4
-  CURL PHP extension (already present on most PHP installations)

### Installation

The preferred installation method is trough [Composer](http://getcomposer.org).

On the command line, on your project's folder, type:

```shell
composer require claudio-silva/http-client
```

## Examples

```php
use Http;
$req = new Client ('http://some.net/api');
```

#### The simplest case: get a text/html page

```php
$text = $req->get ('index.html')->send ();
```

#### More complex requests can be made easily

```php
// Get http://some.net/api/books?id=1
$xml = $req           
  ->get ('books')
  ->param ('id', 1)
  ->expectXml ()
  ->header ('X-Custon', 'test')
  ->send ();

// Get http://some.net/api/books/author1/id1?page=1&filter=john
$book = $req          
  ->get ('books/%s/%s', 'author1', 'id1')
  ->expectJson ()
  ->params ([
    'page' => 1,
    'filter' => 'john'
  ])
  ->headers ([
    'Header-One' => 'some value',
    'Header-Two' => 'some value'
  ])
  ->send ();
```

#### Fully REST

```php
$req->post ('authors/%s/posts', $author)->with ($data)->send ();

$req->put ('authors/%s/posts/%s', $author, $id)->with ($data)->send ();

$req->delete ('posts/%s', $id)->send ();
```

#### Total control is available, if you need it

```php
$req->autoCheck = false;                  //throw no exceptions
$req->referrer = 'http://google.com';
$req->cookieJar ['MyCookie'] = 'my value';
$req->method ('get')
    ->url ('books/%s/%s', $name, $id)
    ->header ('Accept', 'application/json')
    ->transform (function ($response) { return json_decode ($response); })
    ->send ();
if ($req->responseStatus != 200)
  throw new HttpException ("Can't load book", $req->responseStatus);
echo "My session: " . $req->responseCookies['PHPSESSID'];
$book = $req->responseBody;
++$book->reads;
// Begin a new request, on the same browsing context (referrer and cookies are automatically set)
$req
  ->begin ()
  ->put ('books/%s/%s', $name, $id)
  ->with ($book)
  ->send ();
```

## API

#### baseUrl ($url)

Sets the URL prefix for further requests.

```
@param string $url
@return ClientInterface
```

It appends a trailing slash if it is not present.

#### begin ()

Returns a new request based on the current request, keeping the current navigation session.

```
@return ClientInterface
```

The base URL, cookies and referrer are preserved. All other settings are initialized to their default values.

#### delete ($url)

Begins a new request by setting its method to `DELETE` and its URL to the specified one.

```
@param string $url
@param string ...$args URL paramet
@return ClientInterface
```

> It clears the remaining request data (params, headers, etc).
> To issue a new request reusing the current data, call `send()` again.

This method supports parametric URLs (ex: dynamic URL segments).
Any remaining arguments on the method call are injected into the URL on placeholders written in `sprintf` syntax.

> To set URL parameters, use `param()` instead of this.

Ex:
```php
$req->delete ('posts/%s', $id)->send ();
```

#### expectJson ($associative = false)

Sets the `Accept` header to JSON and the response transformer to a JSON parser, for subsequent requests.

```
@param bool $associative When `true`, returned objects will be converted into associative arrays.
@return ClientInterface
```

#### expectText ()

Sets the `Accept` header to the most common text types and the response transformer to `null`, on all future
requests.

```
@return ClientInterface
```

#### expectXml ($fullDOM = false, $options = 0)

Sets the `Accept` header to JSON and the response transformer to a XML parser, for subsequent requests.

```
@param bool $fullDOM When `true`, a DOMDocument object will be returned, SimpleXmlElement otherwise.
@param int  $options Bitwise OR of the libxml option constants.
@return ClientInterface
```

<p>`send()` will return `DOMDocument | SimpleXmlElement | null| false`.
<br>`null` or `false` may mean the document could not be parsed.

#### get ($url)

Begins a new request by setting its method to `GET` and its URL to the specified one.

```
@param string $url
@param string ...$args Remaining arguments are injected into the URL on placeholders with `sprintf` syntax.
@return ClientInterface
```

> It clears the remaining request data (params, headers, etc).
> To issue a new request reusing the current data, call `send()` again.

This method supports parametric URLs (ex: dynamic URL segments).
Any remaining arguments on the method call are injected into the URL on placeholders written in `sprintf` syntax.

> To set URL parameters, use `param()` instead of this.

Ex:
```php
$req->get ('api/%s/%s', $name, $id)->send ();
```

#### header ($name, $value)

Adds a header to the current request.

```
@param string|string[]       $name
@param string|int|float|null $value If null, the header will be removed.
@return ClientInterface
```

#### headers (array $map)

Adds multiple headers to the current request.

```
@param  array $map A map of header names to header values.
@return ClientInterface
@see    HttpRequestInterface::param()
```

#### method ($verb)

Sets the request's HTTP verb.

```
@param string $verb One of `get|put|post|delete|patch|head|options|connect|trace`.
@return ClientInterface
```

#### param ($name, $value)

Adds an URL parameter to the current request.

```
@param string           $name
@param string|int|float $value
@return ClientInterface
```

<p>Multiple parameters with the same name are allowed.
<p>Null values will cause the parameter to NOT be added.
<p>Empty strings will add a parameter with an empty value.


#### params (array $map)

Adds multiple URL parameters to the current request.

```
@param  array $map A map of parameter names to parameter values.
@return ClientInterface
@see    HttpRequestInterface::param()
```

#### post ($url)

Begins a new request by setting its method to `POST` and its URL to the specified one.

```
@param string $url
@param string ...$args Remaining arguments are injected into the URL on placeholders with `sprintf` syntax.
@return ClientInterface
```

> It clears the remaining request data (params, headers, etc).
> To issue a new request reusing the current data, call `send()` again.

This method supports parametric URLs (ex: dynamic URL segments).
Any remaining arguments on the method call are injected into the URL on placeholders written in `sprintf` syntax.

> To set URL parameters, use `param()` instead of this.

Ex:
```php
$req->post ('authors/%s/posts', $author)->with ($data)->send ();
```

#### put ($url)

Begins a new request by setting its method to `PUT` and its URL to the specified one.

```
@param string $url
@param string ...$args Remaining arguments are injected into the URL on placeholders with `sprintf` syntax.
@return ClientInterface
```

> It clears the remaining request data (params, headers, etc).
> To issue a new request reusing the current data, call `send()` again.

This method supports parametric URLs (ex: dynamic URL segments).
Any remaining arguments on the method call are injected into the URL on placeholders written in `sprintf` syntax.

> To set URL parameters, use `param()` instead of this.

Ex:
```php
$req->put ('authors/%s/posts/%s', $author, $id)->with ($data)->send ();
```

#### send ()

Performs the currently chained request and returns the response text.

```
@return mixed The server's response body, eventually transformed.
```

> This method doesn't set the `Accept` header. You should set it previously via the `expectXXX()` methods.

#### transform ($callback)

Sets the response transformer function.

```
@param callable $callback A function to transform the response. It receives the raw response text as argument.
@return ClientInterface
```

#### url ($url)

Sets the URL of the request.

```
@param string $url
@param string ...$args Remaining arguments are injected into the URL on placeholders with `sprintf` syntax.
@return ClientInterface
```

This method supports parametric URLs (ex: dynamic URL segments).
Any remaining arguments on the method call are injected into the URL on placeholders written in `sprintf` syntax.

> To set URL parameters, use `param()` instead of this.

Ex:
```php
$req->method ('get')->url ('api/%s/%s', $name, $id)->send ();
```

#### with ($body, $type = null)

Sets the request payload for POST or PUT requests.

```
@param mixed  $body The data to be sent with the request.
@param string $type One of `json|form|text|xml`. The appropriate `Content-Type` header will be added. If not
                    specified, no header will be set and the body will not be serialized.
@return ClientInterface
```
Depending on the `$type` argument, the body will be serialized accordingly.

## License

The HTTP Client library is open-source software licensed under the [MIT license](http://opensource.org/licenses/MIT).

Copyright &copy;2015 Cláudio Manuel Brás da Silva
