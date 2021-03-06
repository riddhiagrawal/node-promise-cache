![Travis-CI Status](https://travis-ci.org/Vinelab/node-promise-cache.svg?branch=master)

# Promise Cache

A promise-based cache store that preserves stored data types and is easily expandable with custom cache stores. Redis is supported out of the box.

> Inspired by [Laravel's Cache](http://laravel.com/docs/cache)

## Installation

`npm install promise-cache`

## Usage

This code is assumed existing in all of the upcoming examples.

`Cache = require('promise-cache').create()`

##### Storing items in the cache

```javascript
Cache.put('key', 'value' 20); // will be stored for 20 seconds.

Cache.forever('other', 'thing'); // will be stored indefinitely
```

##### Retrieving cached items

```javascript
Cache.get('key').then(function(result){
    console.log(result); // value will be whatever we have stored.
});
```

Since `Cache` works in promises then you may assign the operation to a variable and attach handlers to it:

```javascript
found = Cache.get('key');

found.then(function(value){
    // do things with the value.
});

found.then(function(value){
    // do something else with the value.
});

found.fail(function(why){
    // something went wrong while fetching, see why.
});
```

##### Remembering the results of a closure

Sometimes you may wish to perform a certain operation and store its results if the data you're looking for
was not found in the cache. This can be achieved using `remember` or `rememberForever` methods.

###### Simple use-case
```javascript
Cache.remember('some-key', 22, function(store){
    data = 'do something to get the data';

    store(data);
});
```

###### Promise use-case
```javascript
Http = require('promise-http').client;

cached = Cache.remember('onetwo', 60, function(store){
    request = Http.get('http://echo.jsontest.com/key/value/one/two')
    request.then(function(response){
        // store the JSON representation of the response.
        store(JSON.prase(response));
    });
});

// data will be remembered for 60 seconds and the request will be issued
// again when the time has elapsed.
cached.then(function(data){
    console.log('do things with data');
});
```

The above example with `rememberForever` would store the data forever:

```javascript
Http = require('promise-http').client;

forever = Cache.rememberForever('threefour', function(store){
    request = Http.get('http://echo.jsontest.com/key/value/three/four')
    request.then(function(response){
        // store the JSON representation of the response.
        store(JSON.parse(response));
    });
});

// data will forever be stored (until flushed or removed by someone else)
forever.then(function(data){
    console.log('do things with the data, again.');
});
```

As you can see, regardless of the type of data we are storing, `promise-cache` will make sure to return just what you stored
preserving the data types for you - all data types are supported.

## Configuration

You may configure the redis store or specify another store you have built yourself.
This package uses [redis-pool](https://www.npmjs.org/package/sol-redis-pool) to manage Redis connections.

> The values listed below are the defaults.

```javascript
Cache = require('promise-cache').create({
    prefix: 'pcache',
    ttl: 60,
    redis: {
        host: '127.0.0.1',
        port: 6379,
        pool:{
            max: 1,
            min: 1
        }
    }
});
```

You may also have different instances connected to different stores.

```javascript
LocalCache = require('promise-cache').create();

CentralCache = require('promise-cache').create({
    redis: {host: '192.168.53.69'}
});
```

Custom store: check the [Custom Store section](#custom-store)

```javascript
Cache = require('promise-cache').create({
    store: './stores/RedisStore' // set the /path/to/your/store
});
```

## Methods

### getCacheKey(key)

Get the cache key.

#### Params:

* **mixed** *key*

#### Return:

* **string**

### put(key, value, ttl)

Store an item in the cache for a given number of seconds.

#### Params:

* **mixed** *key*
* **mixed** *value*
* **int** *ttl*

#### Return:

* **q.promise** Deferred Promise.

### forever(key, value)

Store an item in the cache indefinitely.

#### Params:

* **mixed** *key*
* **mixed** *value*

#### Return:

* **q.promise**

### remember(key, closure)

Get an item from cache if it exists,
otherwise call `closure` and store the results.

#### Params:

* **mixed** *key*
* **function** **closure**

### rememberForever(key, closure)

Just like remember(), get an item from the cache
or store the resulting value from the promise.

#### Params:

* **mixed** *key*
* **function** *closure*

### has(key)

Determine if an item exists in the cache.

#### Params:

* **mixed** *key*

#### Return:

* **q.promise**

### increment(key, count)

Increment the value of an item in the cache.

#### Params:

* **mixed** *key*
* **int** *count* (Default 1)

#### Return:

* **q.promise**

### decrement(key, count)

Decrement the value of an item in the cache.

#### Params:

* **mixed** *key*
* **int** *count* (Default 1)

#### Return:

* **q.promise**

### ttl(key)

Get the remaining TTL (Time To Live) of an item in the cache.

#### Params:

* **mixed** *key*

### get(key)

Retrieve an item from the cache by key.

#### Params:

* **mixed** *key*

#### Return:

* **q.promise**

### forget(key)

Remove an item from the cache by key.

#### Params:

* **mixed** *key*

#### Return:

* **q.promise**

### flush()

Remove all items from the cacheKey

#### Return:

* **q.promise**

## Custom Store

To implement your own store, the following methods needs to be implemented.

- `cb` is the callback function that needs to be called with `(error, result)`
    - `error` should be `null` in the case of no errors. Otherwise the cache promise will be rejected.
    - `result` is the result of the operation.

### set(key, value, cb)

Set an item by key.

#### Params:

* **string** *key*
* **string** *value*
* **function** *cb*

### setFor(key, value, ttl, cb)

Set an item by key for a certain number of minutes.

#### Params:

* **string** *key*
* **string** *value*
* **int** *ttl*
* **function** *cb*

### has(key, cb)

Determine whether an item exists in the store.

#### Params:

* **string** *key*
* **function** *cb*

### ttl(key, cb)

Get the TTL of an item.

#### Params:

* **string** *key*
* **function** *cb*

### get(key, cb)

Get an item from the store.

#### Params:

* **string** *key*
* **function** *cb*

### del(key, cb)

Delete an item from the store.

#### Params:

* **string** *key*
* **function** *cb*

### flush(cb)

Flush the store.

#### Params:

* **function** *cb*

### increment(key, count, cb)

Increment the integer value of a key by @count. Default is 1.

#### Params:

* **string** *key*
* **int** *count*
* **function** *cb*

### decrement(key, count, cb)

Decrement the integer value of a key by @count. Default is 1.

#### Params:

* **string** *key*
* **int** *count*
* **funciton** *cb*

## Tests

The tests for this package are divided into two. One in `spec/` and the other is in `test/` where they represent
**unit testing** and **functional testing** respectively.

- First: `npm install`
- Running specs: `npm run-script spec`
- Running tests:
    - Make sure to have a Redis instance running if you're using the default store. i.e. `docker run -d -p 6379:6379 redis`
    - if configuration is needed set it up in `test/config.coffee`
    - `npm run-script test`

