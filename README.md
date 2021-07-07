# Intro

An introduction to the Web Straight Line

## What is the Web Straight Line

The Web Straight Line is a new design paradigm for web framework development, It considers web process as a line with many nodes.

These nodes can be called web logic. Different requests may have different/same nodes, but all nodes are sequential in one request.

So we can call the request Web Straight Lines.

## Why Web Straight Line instead of MVC?

Web Straight Line calls for deprecating the MVC paradigm.
Becuase Web Logic are mostly about data input / output / processing, request routing and access control, it is not about user interface, there is no MVC applicable.

## The Web Staight Line Diagram

Most Web Straight Line process can be describe by one Diagram:

![Web_Straight_Line](https://user-images.githubusercontent.com/131776/124710884-f9a92180-df2f-11eb-8beb-d4e8f41aef20.png)


## The Philosophy Behide the Web Staight Line

1. The baisc web logic will not change in forseeable time.
2. Web logic and bussiness logic should not be mixtured in a framework.
3. A web framework should only focus on Web Logic.
4. Detailize the repeated web logics and make them easy to be adopted into real code.
5. Web Logic is not about user interface, there is no MVC applicable.
6. Web Logic is only about data input / output / processing, request routing and access control.
7. It is encouraged for framework to provide injectable code to embed web logic instruction into bussiness logic code.
8. Coexsitable with different programming methods(OOP, AOP, FP, etc)

## Implementation

Web Staight Line should be able to be implemented with different languages in different mananers.


Here is a example of how to apply WSL to Typescript + node: 


## [aex](https://www.npmjs.com/package/@aex/core) (Web Straight Line Implementation in node.js)


### 1. HTTP method decorators

This decorators are the most basic decorators, all decorators should follow them. They are
`@http` , `@get` , `@post` .

#### `@http` , `@get` , `@post`

`@http` is the generic http method decorator. `@get` , `@post` are the shortcuts for `@http` ;

The `@http` decorator defines your http handler with a member function.

The member methods are of `IAsyncMiddleware` type.

`@http` takes two parameter:

1. http method name(s)
2. url(s);

You can just pass url(s) if you use http `GET` method only or you can use `@get` .

Here is how your define your handlers.

```ts
import { http, get, post } from "@aex/core";

class User {
  @http("get", ["/profile", "/home"])
  profile() {}

  @http(["get", "post"], "/user/login")
  login() {}

  @http("post", "/user/logout")
  logout() {}

  @http("/user/:id")
  info() {}

  @http(["/user/followers", "/user/subscribes"])
  followers() {}

  @get(["/user/get", "/user/gets"])
  rawget() {}

  @post("/user/post")
  rawpost() {}
}
```

### 2. Data parsing decorators

These decorators will parse all data passed thought the HTTP protocol.
They are `@formdata` , `@query` , `@body` .

1. `@formdata` can parse `mulit-part` formdata such as files into `scope.files` and other formdata into `scope.body`. When parsed, you can retrieve your `multi-part` formdata from `scope.files`, `scope.body`.
2. `@query` can parse url query into `scope.query`.
3. `@body` can parse some simple formdata into `scope.body`.

#### `@formdata`

Decorator `@formdata` is a simplified version of node package [`busboy`](https://github.com/mscdex/busboy) for `aex` , only the `headers` options will be auto replaced by `aex` . So you can parse valid options when necesary.
All uploaded files are in array format, and it parses body as well.

```ts
import { http, formdata } from "@aex/core";

class Formdata {
  protected name = "formdata";

  @http("post", "/file/upload")
  @formdata()
  public async upload(_req: any, res: any, scope: any) {
    const { files, body } = scope;

    // Access your files
    const uploadedSingleFile = files["fieldname1"][0];
    const uploadedFileArray = files["fieldname2"];

    // Access your file info

    uploadedSingleFile.temp; // temporary file saved
    uploadedSingleFile.filename; // original filename
    uploadedSingleFile.encoding; // file encoding
    uploadedSingleFile.mimetype; // mimetype

    // Access none file form data
    const value = body["fieldname3"];
    res.end("File Uploaded!");
  }
}
```

#### `@body`

Decorator @body provides a simple way to process data with body parser. It a is a simplified version of node package [body-parser](https://github.com/expressjs/body-parser).

It takes two parameters:

1. types in ["urlencoded", "raw", "text", "json"]
2. options the same as body-parser take.

then be parsed into `scope.body` , for compatibility `req.body` is still available.

Simply put:

```ts
@body("urlencoded", { extended: false })
```

Full example

```ts
import { http, body } from "@aex/core";

class User {
  @http("post", "/user/login")
  @body("urlencoded", { extended: false })
  login() {
    const [, , scope] = arguments;
    const { body } = scope;
  }

  @http("post", "/user/logout")
  @body()
  login() {
    const [, , scope] = arguments;
    const { body } = scope;
  }
}
```

#### `@query`

Decorator @query will parse query for you. After decorated with `@query` you will have `scope.query` to use. `req.query` is available for compatible reasion, but it is discouraged.

```ts
class Query {
  @http("get", "/profile/:id")
  @query()
  public async id(req: any, res: any, _scope: any) {
    // get /profile/111?page=20
    req.query.page;
    // 20
  }
}
```

### 3. Static file serving decorators

Aex provides `@serve` decorator and its alias `@assets` for static file serving.

> Due to `static` is taken as a reserved word for javascript, `static` is not supported.

#### `@serve` and `@assets`

They take only one parameter:

1. url: the base url for your served files.

It is recursive, so place used with caution, don't put your senstive files under that folder.

```ts
import { serve } from "@aex/core";

class StaticFileServer {
  protected name = "formdata";

  @serve("/assets")
  public async upload() {
    // All your files and subdirectories are available for accessing.
    return resolve(__dirname, "./fixtures");
  }

  @assets("/assets1")
  public async upload() {
    // All your files and subdirectories are available for accessing.
    return resolve(__dirname, "./fixtures");
  }
}
```

### 4. Session management decorators

Aex provides `@session` decorator for default cookie based session management.
Session in other format can be realized with decorator `@inject` .

#### `@session`

Decorator `@session` takes a store as the parameter. It is an object derived from the abstract class ISessionStore. which is defined like this:

```ts
export declare abstract class ISessionStore {
  abstract set(id: string, value: any): any;
  abstract get(id: string): any;
  abstract destroy(id: string): any;
}
```

`aex` provides two default store: `MemoryStore` and `RedisStore` .
`RedisStore` can be configurated by passing options through its constructor. The passed options is of the same to the function `createClient` of the package `redis` . You can check the option details [here](https://github.com/NodeRedis/node-redis#options-object-properties)

For `MemoryStore` , you can simply decorate with `@session()` .
For `RedisStore` , you can decorate with an RedisStore as `@session(redisStore)` . Be sure to keep the variable redisStore global, because sessions must share only one store.

```ts
// Must not be used @session(new RedisStore(options)).
// For sessions share only one store over every request.
// There must be only one object of the store.
const store = new RedisStore(options);
class Session {
  @post("/user/login")
  @session()
  public async get() {
    const [, , scope] = arguments;
    const { session } = scope;
    session.user = user;
  }

  @get("/user/profile")
  @session()
  public async get() {
    const [, , scope] = arguments;
    const { session } = scope;
    const user = session.user;
    res.end(JSON.stringify(user));
  }

  @get("/user/redis")
  @session(store)
  public async get() {
    const [, , scope] = arguments;
    const { session } = scope;
    const user = session.user;
    res.end(JSON.stringify(user));
  }
}
```

> Share only one store object over requests.

### 5. Data filtering and validation decorators

Aex provides `@filter` to filter and validate data for you.

#### `@filter`

Decorator `@filter` will filter `body` , `params` and `query` data for you, and provide fallbacks respectively for each invalid data processing.

If the filtering rules are passed, then you will get a clean data from `scope.extracted`.

You can access `scope.extracted.body`, `scope.extracted.params` and `scope.extracted.query` if you filtered them.

But still you can access `req.body`, `req.query`, `req,params` after filtered.

Reference [node-form-validator](https://github.com/calidion/node-form-validator) for detailed usage.

```ts
class User {
  private name = "Aex";
  @http("post", "/user/login")
  @body()
  @filter({
    body: {
      username: {
        type: "string",
        required: true,
        minLength: 4,
        maxLength: 20
      },
      password: {
        type: "string",
        required: true,
        minLength: 4,
        maxLength: 64
      }
    },
    fallbacks: {
      body: async(error, req, res, scope) {
        res.end("Body parser failed!");
      }
    }
  })
  public async login(req: any, res: any, scope: any) {
    // req.body.username
    // req.body.password
    // scope.extracted.body.username
    // scope.extracted.body.password
    // scope.extracted is the filtered data
  }

  @http("get", "/profile/:id")
  @body()
  @query()
  @filter({
    query: {
      page: {
        type: "numeric",
        required: true
      }
    },
    params: {
      id: {
        type: "numeric",
        required: true
      }
    },
    fallbacks: {
      params: async function (this: any, _error: any, _req: any, res: any) {
        this.name = "Alice";
        res.end("Params failed!");
      },
    }
  })
  public async id(req: any, res: any, _scope: any) {
    // req.params.id
    // req.query.page
  }
}
```

### 6. Error definition decorators

Aex provides `@error` decorator for error definition

#### @error

Decorator `@error` will generate errors for you.

Reference [errorable](!https://github.com/calidion/errorable) for detailed usage.

`@error` take two parameters exactly what function `Generator.generate` takes.

Besides you can add `lang` attribut to `@error` to default the language, this feature will be automatically removed by `aex` when generate errors.
With the lang attribute, you can new errors without specifying a language every time throw/create an error;

```ts
class User {
  @http("post", "/error")
  @error({
    lang: "zh-CN",
    I: {
      Love: {
        You: {
          code: 1,
          messages: {
            "en-US": "I Love U!",
            "zh-CN": "我爱你！",
          },
        },
      },
    },
    Me: {
      alias: "I",
    },
  })
  public road(_req: any, res: any, scope: any) {
    const [, , scope] = arguments;
    const { error: e } = scope;
    const { ILoveYou } = e;
    throw new ILoveYou('en-US');
    throw new ILoveYou('zh-CN');
    throw new ILoveYou();   // You can ignore language becuase you are now use the default language.
    res.end("User Error!");
  }
}
```
