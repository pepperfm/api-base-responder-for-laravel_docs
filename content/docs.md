---
title: Documentation
description: Standardizing API Responses Without Traits
---

# Docs page

# Problem

I've noticed that most libraries created for API responses are implemented using traits, and the rest are large libraries. These traits implement methods for everything imaginable (response, accepted, created, forbidden...). :br

As a result, if my controller only has 1-2 methods, including such a trait brings a lot of unnecessary clutter into the class. In a couple of large libraries with 700+ stars, I see overengineering at the UX level (from my perspective as a user of the library). :br

### Solution

Write my own library! :br

I decided to create a data processing logic that would require:

- minimal actions at the user level
- simplicity of use
- readability

That is, to get a standardized response, all we need to do is return a response via the library object: :br

```js
composer require pepperfm/api-responder-for-laravel
```

:br

As a result, the basic minimum we get right after installing the library is:

Successful response:

```json
{
  "response": {
    "data": {
      "entities": []|{},
      "meta": []|{},
      "message": "Success"
    }
  }
}
```

Error response:

```json
{
  "response": {
    "data": {
      "errors": null,
      "message": "Error"
    }
  }
}
```

:br

#### Usage examples

````md
```php
public function __construct(public ResponseContract $json)
{
}

public function index(Request $request)
{
    $users = User::query()->get();

    return $this->json->response($users);
}

public function store(UserService $service)
{
    try {
        app('db')->beginTransaction();

        $service->update(request()->input());
        
        app('db')->commit();
    } catch (\Exception $e) {
        app('db')->rollback();
        logger()->debug($e->getMessage());

        return $this->json->error(
            message: $e->getMessage(),
            httpStatusCode: $e->getCode()
        );
    }

    return $this->json->response($users);
}
```
````

As a result, with a successful response, we have the format unpacked as: `response.data.entities`. By default, the format is relevant in the context of REST, so for the `show()` and `update()` methods, the response will be in the format: `response.data.entity`. :br

## Deep Dive

Of course, for customization enthusiasts and configuration explorers, I also created a code sandbox to play around with. :br

### Features

#### Our Favorite Shugar

A wrapper over the `response()` method for pagination:

````md
```php
/*
 * Generate response.data.meta.pagination from first argument of `paginated()` method
 */
public function index(Request $request)
{
  $users = User::query()->paginate();

  return $this->json->paginated($users);
}
```
````

:br

The `paginated()` method accepts two main parameters:

````md
```php
array|\Illuminate\Pagination\LengthAwarePaginator $data,
array|\Illuminate\Pagination\LengthAwarePaginator $meta = [],
```
````

In its logic, it resolves them and adds them to the response under the `meta` key — pagination key.

Response interfaces according to the format returned by Laravel:

```ts
export interface IPaginatedResponse<T> {
    current_page: number
    per_page: number
    last_page: number
    data: T[]
    from: number
    to: number
    total: number
    prev_page_url?: any
    next_page_url: string
    links: IPaginatedResponseLinks[]
}
export interface IPaginatedResponseLinks {
    url?: any
    label: string
    active: boolean
}
```

As a result, the response looks like:

```json
{
  "response": {
    "data": {
      "entities": []|{},
      "meta": {
        "pagination": IPaginatedResponse<T>
      },
      "message": "Success"
    }
  }
}
```

:br

A wrapper over the `response()` method for status codes:

````md
```php
public function store(UserService $service)
{
    // message: 'Stored', httpStatusCode: JsonResponse::HTTP_CREATED
    return $this->json->stored();
}

public function destroy()
{
    // message: 'Deleted', httpStatusCode: JsonResponse::HTTP_NO_CONTENT
    return $this->json->deleted();
}
```
````

:br

#### Working with Different Parameter Types

The first argument of the response() method can be of types array|Arrayable, so data can be mapped before passing to the method within these types. For example:

````md
```php
public function index()
{
    $users = User::query()->paginate();
    $dtoCollection = $users->getCollection()->mapInto(UserDto::class);

    return resolve(ResponseContract::class)->paginated(
        data: $dtoCollection,
        meta: $users
    );
}
```

```php
public function index()
{
    $users = SpatieUserData::collect(User::query()->get());

    return \ApiBaseResponder::response($users);
}
```
````

:br

#### Customization via Config

The config itself:

````md
```php
return [
    /*
    |--------------------------------------------------------------------------
    | Plural Data Key
    |--------------------------------------------------------------------------
    |
    | This key represents the plural form of data entities in the response.
    | It is used when returning collections of resources.
    |
    */

    'plural_data_key' => 'entities',

    /*
    |--------------------------------------------------------------------------
    | Singular Data Key
    |--------------------------------------------------------------------------
    |
    | This key represents the singular form of a data entity in the response.
    | It is used when returning a single resource.
    |
    */

    'singular_data_key' => 'entity',

    /*
    |--------------------------------------------------------------------------
    | Use RESTful Format
    |--------------------------------------------------------------------------
    |
    | Enable this option to follow RESTful conventions in response formatting.
    | If set to true, responses will adhere to standard REST API practices.
    |
    */

    'using_for_rest' => true,

    /*
    |--------------------------------------------------------------------------
    | Methods for Singular Response
    |--------------------------------------------------------------------------
    |
    | Specify the HTTP methods that should return data in singular format.
    | Add additional methods as necessary.
    |
    */

    'methods_for_singular_key' => ['show', 'update'],

    /*
    |--------------------------------------------------------------------------
    | Force JSON Response Header
    |--------------------------------------------------------------------------
    |
    | When enabled, this option ensures that the response includes a JSON
    | content type header, signaling to clients that the response is in JSON.
    |
    */

    'force_json_response_header' => true,

    /*
    |--------------------------------------------------------------------------
    | Disable wrapping of the outer-most resource array.
    |--------------------------------------------------------------------------
    |
    | Enable this option to return response without data_key wrapping.
    |
    */

    'without_wrapping' => false,
];
```
````

Disabling `using_for_rest` keeps the returned format always `response.data.entities` (plural) regardless of the method from which the call is made. :br

With `methods_for_singular_key`, you can add to the list of methods where the key will be returned in the singular. `force_json_response_header` essentially adds the header to requests in the classic way: `$request->headers->set('Accept', 'application/json');`

#### Customization via Attributes

Blocking `using_for_rest` and `methods_for_singular_key` values in the config to set the response key according to `singular_data_key`:

````md
```php
#[ResponseDataKey]
public function attributeWithoutParam(): JsonResponse
{
    // response.data.entity
    return BaseResponse::response($this->user);
}
```
````

Similarly, you can pass your own key name:

````md
```php
#[ResponseDataKey('random_key')]
public function attributeWithParam(): JsonResponse
{
    // response.data.random_key
    return BaseResponse::response($this->user);
}
```
````

### In conclusion

The main need is covered: I wanted to be able to simply install the library and have a concise basis for standardizing the response format out of the box. Without unnecessary movements. :br

And of course, there are still plenty of opportunities for customization and additional sugar, so further development of the library lies ahead) but the main idea will definitely be preserved.
