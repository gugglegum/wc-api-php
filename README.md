# WooCommerce API - PHP Client

A PHP wrapper for the WooCommerce REST API. Easily interact with the WooCommerce REST API securely using this library. If using a HTTPS connection this library uses BasicAuth, else it uses Oauth to provide a secure connection to WooCommerce.

[![build status](https://secure.travis-ci.org/woocommerce/wc-api-php.svg)](http://travis-ci.org/woocommerce/wc-api-php)
[![Scrutinizer Code Quality](https://scrutinizer-ci.com/g/woocommerce/wc-api-php/badges/quality-score.png?b=master)](https://scrutinizer-ci.com/g/woocommerce/wc-api-php/?branch=master)
[![PHP version](https://badge.fury.io/ph/automattic%2Fwoocommerce.svg)](https://packagist.org/packages/automattic/woocommerce)

## !!! What this fork does? !!!

Some web-servers restrict HTTP methods to only GET and POST. Woo-Commerce uses PUT and DELETE in its API. But there are 2 workarounds for such servers:
1. Use the POST method and add the "_method=" query parameter to the URL using the real HTTP method.
2. Use the POST method and add the "X-HTTP-Method-Override" HTTP header with real HTTP method to the request.

Both variants are equivalent. And they can even be used together. This fork adds possibility to use these workarounds.

More information on this can be found here:

 * https://developer.wordpress.org/rest-api/using-the-rest-api/global-parameters/#_method-or-x-http-method-override-header
 * https://gridpane.com/kb/put-requests-woocommerce-api/
 * https://gridpane.com/kb/making-nginx-accept-put-delete-and-patch-verbs/
 * https://github.com/woocommerce/woocommerce/issues/15218

All you need to do to activate workaround is to add 1 or 2 options to constructor of the Client like in this example:

```php
use Automattic\WooCommerce\Client;

$client = new Client($url, $consumerKey, $consumerSecret, [
    'wp_api' => true,
    'version' => 'wc/v3',
    'follow_redirects' => true,
    'query_string_auth' => true,
    'timeout' => 60,
    'method_override_query' => true,
    'method_override_header' => false,
]);
```

The options `method_override_query` activates `?_method=PUT/DELETE` workaround. And the option `method_override_header` activates workaround with `X-HTTP-Method-Override` header. Usually it's enough to activate only one of them, but you can activate they both if you want.

If you want to better understand how it's working on Woo-Commerce side, you can review the code in the file `class-wc-api-server.php`. At the moment this code looks like this:

```php
// Compatibility for clients that can't use PUT/PATCH/DELETE
if ( isset( $_GET['_method'] ) ) {
    $this->method = strtoupper( $_GET['_method'] );
} elseif ( isset( $_SERVER['HTTP_X_HTTP_METHOD_OVERRIDE'] ) ) {
    $this->method = $_SERVER['HTTP_X_HTTP_METHOD_OVERRIDE'];
}
```

## Installation

Since this is a simple fork of the original package (without changing the Composer package name and registering it on Packagist.org), the installation is little more complicated than simply adding of one line into the `"require"` section. You need to override the location of this package in order to install it from my fork. So, basically you need to combine your `composer.json` with following object:

```
{
  "repositories": [
    {
        "type": "vcs",
        "url": "git@github.com:gugglegum/wc-api-php.git"
    }
  ],
  "require": {
    "automattic/woocommerce": "^3.0",
  }
}

```

And then run `composer update`.

## Getting started

Generate API credentials (Consumer Key & Consumer Secret) following this instructions <http://docs.woocommerce.com/document/woocommerce-rest-api/>
.

Check out the WooCommerce API endpoints and data that can be manipulated in <https://woocommerce.github.io/woocommerce-rest-api-docs/>.

## Setup

Setup for the new WP REST API integration (WooCommerce 2.6 or later):

```php
require __DIR__ . '/vendor/autoload.php';

use Automattic\WooCommerce\Client;

$woocommerce = new Client(
    'http://example.com', 
    'ck_XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX', 
    'cs_XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX',
    [
        'version' => 'wc/v3',
    ]
);
```

## Client class

```php
$woocommerce = new Client($url, $consumer_key, $consumer_secret, $options)
```

### Options

| Option            | Type     | Required | Description                                |
|-------------------|----------|----------|--------------------------------------------|
| `url`             | `string` | yes      | Your Store URL, example: http://woo.dev/   |
| `consumer_key`    | `string` | yes      | Your API consumer key                      |
| `consumer_secret` | `string` | yes      | Your API consumer secret                   |
| `options`         | `array`  | no       | Extra arguments (see client options table) |

#### Client options

| Option              | Type     | Required | Description                                                                                                            |
|---------------------|----------|----------|------------------------------------------------------------------------------------------------------------------------|
| `version`           | `string` | no       | API version, default is `wc/v3`                                                                                        |
| `timeout`           | `int`    | no       | Request timeout, default is `15`                                                                                       |
| `verify_ssl`        | `bool`   | no       | Verify SSL when connect, use this option as `false` when need to test with self-signed certificates, default is `true` |
| `follow_redirects`  | `bool`   | no       | Allow the API call to follow redirects                                                                                 |
| `query_string_auth` | `bool`   | no       | Force Basic Authentication as query string when `true` and using under HTTPS, default is `false`                       |
| `oauth_timestamp`   | `string` | no       | Custom oAuth timestamp, default is `time()`                                                                            |
| `user_agent`        | `string` | no       | Custom user-agent, default is `WooCommerce API Client-PHP`                                                             |
| `wp_api_prefix`     | `string` | no       | Custom WP REST API URL prefix, used to support custom prefixes created with the `rest_url_prefix` filter               |
| `wp_api`            | `bool`   | no       | Set to `false` in order to use the legacy WooCommerce REST API (deprecated and not recommended)                        |

## Client methods

### GET

```php
$woocommerce->get($endpoint, $parameters = [])
```

### POST

```php
$woocommerce->post($endpoint, $data)
```

### PUT

```php
$woocommerce->put($endpoint, $data)
```

### DELETE

```php
$woocommerce->delete($endpoint, $parameters = [])
```

### OPTIONS

```php
$woocommerce->options($endpoint)
```

#### Arguments

| Params       | Type     | Description                                                  |
|--------------|----------|--------------------------------------------------------------|
| `endpoint`   | `string` | WooCommerce API endpoint, example: `customers` or `order/12` |
| `data`       | `array`  | Only for POST and PUT, data that will be converted to JSON   |
| `parameters` | `array`  | Only for GET and DELETE, request query string                |

#### Response

All methods will return arrays on success or throwing `HttpClientException` errors on failure.

```php
use Automattic\WooCommerce\HttpClient\HttpClientException;

try {
    // Array of response results.
    $results = $woocommerce->get('customers');
    // Example: ['customers' => [[ 'id' => 8, 'created_at' => '2015-05-06T17:43:51Z', 'email' => ...
    echo '<pre><code>' . print_r( $results, true ) . '</code><pre>'; // JSON output.

    // Last request data.
    $lastRequest = $woocommerce->http->getRequest();
    echo '<pre><code>' . print_r( $lastRequest->getUrl(), true ) . '</code><pre>'; // Requested URL (string).
    echo '<pre><code>' . print_r( $lastRequest->getMethod(), true ) . '</code><pre>'; // Request method (string).
    echo '<pre><code>' . print_r( $lastRequest->getParameters(), true ) . '</code><pre>'; // Request parameters (array).
    echo '<pre><code>' . print_r( $lastRequest->getHeaders(), true ) . '</code><pre>'; // Request headers (array).
    echo '<pre><code>' . print_r( $lastRequest->getBody(), true ) . '</code><pre>'; // Request body (JSON).

    // Last response data.
    $lastResponse = $woocommerce->http->getResponse();
    echo '<pre><code>' . print_r( $lastResponse->getCode(), true ) . '</code><pre>'; // Response code (int).
    echo '<pre><code>' . print_r( $lastResponse->getHeaders(), true ) . '</code><pre>'; // Response headers (array).
    echo '<pre><code>' . print_r( $lastResponse->getBody(), true ) . '</code><pre>'; // Response body (JSON).

} catch (HttpClientException $e) {
    echo '<pre><code>' . print_r( $e->getMessage(), true ) . '</code><pre>'; // Error message.
    echo '<pre><code>' . print_r( $e->getRequest(), true ) . '</code><pre>'; // Last request data.
    echo '<pre><code>' . print_r( $e->getResponse(), true ) . '</code><pre>'; // Last response data.
}
```

## Release History

- 2019-01-16 - 3.0.0 - Legacy API turned off by default, and improved JSON error handler.
- 2018-03-29 - 2.0.1 - Fixed fatal errors on `lookForErrors`.
- 2018-01-12 - 2.0.0 - Responses changes from arrays to `stdClass` objects. Added `follow_redirects` option.
- 2017-06-06 - 1.3.0 - Remove BOM before decoding and added support for multi-dimensional arrays for oAuth1.0a.
- 2017-03-15 - 1.2.0 - Added `user_agent` option.
- 2016-12-14 - 1.1.4 - Fixed WordPress 4.7 compatibility.
- 2016-10-26 - 1.1.3 - Allow set `oauth_timestamp` and improved how is handled the response headers.
- 2016-09-30 - 1.1.2 - Added `wp_api_prefix` option to allow custom WP REST API URL prefix.
- 2016-05-10 - 1.1.1 - Fixed oAuth and error handler for WP REST API.
- 2016-05-09 - 1.1.0 - Added support for WP REST API, added method `Automattic\WooCommerce\Client::options` and fixed multiple headers responses.
- 2016-01-25 - 1.0.2 - Fixed an error when getting data containing non-latin characters.
- 2016-01-21 - 1.0.1 - Sort all oAuth parameters before build request URLs.
- 2016-01-11 - 1.0.0 - Stable release.
