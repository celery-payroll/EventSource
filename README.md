# EventSource

A PHP 5.3 library for creating an [EventSource](http://dev.w3.org/html5/eventsource/) stream.

EventSource or Server-Sent-Events is a W3C specification that defines a protocol and an API
for pushing data from the server to the client. This library is a server-side implementation
of this protocol.

It is designed to be transport agnostic, allowing you to use it with apache directly or with
other webservers, such as mongrel2.

[![Build Status](https://secure.travis-ci.org/igorw/EventSource.png)](http://travis-ci.org/igorw/EventSource)

## Fetch

The recommended way to install EventSource is [through composer](http://packagist.org).

Just create a composer.json file for your project:

```JSON
{
    "require": {
        "igorw/event-source": "*"
    }
}
```

And run these two commands to install it:

    $ wget http://getcomposer.org/composer.phar
    $ php composer.phar install

Now you can add the autoloader, and you will have access to the library:

```php
<?php
require 'vendor/.composer/autoload.php';
```

## Usage

The first thing you need to do is output the EventSource headers, so that the
client it's talking to an EventSource server.

```php
<?php

use Igorw\EventSource\Stream;

foreach (Stream::getHeaders() as $name => $value) {
    header("$name: $value");
}
```

After that you create a ``Stream`` which provides a nice API for creating events.
Once you call flush, all queued events are sent to the client.

This example will send a new event every 2 seconds.

```php
<?php

use Igorw\EventSource\Stream;

$stream = new Stream();

while (true) {
    $stream
        ->event()
            ->setData("Hello World")
        ->end()
        ->flush();
    
    sleep(2);
}
```

And an example JavaScript client:

```JavaScript
var stream = new EventSource('stream.php');

stream.addEventListener('message', function (event) {
    console.log(event.data);
});
```

## Advanced

### Last event id

When your events have ids, the client will send a `Last-Event-ID` header on
reconnection. You can read this value and re-send any events that occured after
the one provided by the user.

```php
<?php

$lastId = filter_input(INPUT_SERVER, 'HTTP_LAST_EVENT_ID');

if ($lastId) {
    $buffer = getMessagesAfter($lastId);

    foreach ($buffer as $message) {
        $stream->event()
            ->setId($message['id'])
            ->setData($message['data']);
    }

    $stream->flush();
}
```

### Event namespacing

You can namespace events by using the `setEvent` method on the event. This
allows you to bind to those event types specifically on the client side.

Here is a stream that sends two events. One of type `foo` and one of type
`bar`.

```php
<?php

$stream
    ->event()
        ->setEvent('foo')
        ->setData($message['data']);
    ->end()
    ->event()
        ->setEvent('bar')
        ->setData($message['data']);
    ->end()
    ->flush();
```

On the client you bind to that event instead of the generic `message` one.
Do note that the `message` event will not catch these messages.

```JavaScript
var stream = new EventSource('stream.php');

stream.addEventListener('foo', function (event) {
    console.log('Received event foo!');
});

stream.addEventListener('bar', function (event) {
    console.log('Received event bar!');
});
```

### Custom handler

By default the library will assume you are running in a traditional apache-like
environment. This means that output happens through echo. If you are using a
server that handles web output in a different way (eg. app server), then you
will want to change this.

A handler is simply a function that takes a chunk (a single event) and sends it
to the client. You can define it as a lambda. Here is the default handler:

```php
<?php

$handler = function ($chunk) {
    echo $chunk;
    ob_flush();
    flush();
};
```

You just pass it to the constructor of the stream:

```php
<?php

$stream = new Stream($handler);
```

### PHP time limit

In some setups it may be required to remove the time limit of the script and
increase th memory limit. If you are having problems with your script dying
after 30 or 60 seconds, add these two lines.

```php
<?php
set_time_limit(0);
ini_set('memory_limit', '512M');
```

### Polyfill

Most old browsers have not implemented EventSource yet. Luckily there is a
[polyfill](https://github.com/Yaffle/EventSource) available, that allows them
to be used in a wider range of browsers.

Tests
-----

Before running the tests you need to have composer set up an autoloader:

    $ wget http://getcomposer.org/composer.phar
    $ php composer.phar install

Now you can run the unit tests.

    $ phpunit

License
-------
MIT, see LICENSE.
