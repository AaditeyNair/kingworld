# 🦊 KingWorld
Fast, and friendly [Bun](https://bun.sh) web framework.

Focusing on **speed**, and **simplicity**.

###### Named after my favorite VTuber (Shirakami Fubuki) and composer (Sasakure.UK) song [KINGWORLD/白上フブキ(Original)](https://youtu.be/yVaQpUUAzik)

## Feature
- Speed - Build for speed and optimized for Bun in mind.
- Scalable - Designed for micro-service, decoupled logic and treat everything as building block
- Simplicity - Composed patterns into plugin, removing redundant logic into one simple plugin
- Friendliness - Familiar pattern with enhance TypeScript supports eg. auto infers type paramters

⚡️ KingWorld is [one of the fastest Bun web framework](https://github.com/SaltyAom/bun-http-framework-benchmark)

## Ecosystem
KingWorld can be heavily customized with the use of plugins.

Official plugins:
- [Static](https://github.com/saltyaom/kingworld-static) for serving static file/folders
- [Cookie](https://github.com/saltyaom/kingworld-cookie) for reading/setting cookie
- [Schema](https://github.com/saltyaom/kingworld-schema) for validating request declaratively
- [CORS](https://github.com/saltyaom/kingworld-cors) for handling CORs request

## Quick Start
KingWorld is a web framework based on [Bun](https://bun.sh).

```bash
bun add kingworld
```

Now create `index.ts`, and place the following:
```typescript
import KingWorld from 'kingworld'

new KingWorld()
    .get("/", () => "🦊 Now foxing")
    .listen(3000)
```

And run the server:
```bash
bun index.ts
```

Then simply open `http://localhost:3000` in your browser.

Congrats! You have just create a new web server in KingWorld 🎉🎉

## Routing
Common HTTP methods have a built-in methods for convenient usage:
```typescript
app.get("/hi", () => "Hi")
    .post("/hi", () => "From Post")
    .put("/hi", () => "From Put")
    .on("M-SEARCH", async () => "Custom Method")
    .listen(3000)

// [GET] /hi => "Hi"
// [POST] /hi => "From Post"
// [PUT] /hi => "From Put"
// [M-SEARCH] /hi => "Custom Method"
```

To return JSON, simply return any serializable object:
```typescript
app.get("/json", () => ({
    hi: 'KingWorld'
}))

// [GET] /json => {"hi": "KingWorld"}
```

All values returned from handler will be transformed into `Response`.

You can return `Response` if you want to declaratively control the response.
```typescript
app
    .get("/number", () => 1)
    .get("/boolean", () => true)
    .get("/promise", () => new Promise((resovle) => resolve("Ok")))
    .get("/response", () => new Response("Hi", {
        status: 200,
        headers: {
            "x-powered-by": "KingWorld"
        }
    }))

// [GET] /number => "1"
// [GET] /boolean => "true"
// [GET] /promise => "Ok"
// [GET] /response => "Hi"
```

You can use `ctx.status` to explictly set status code without creating `Response`
```typescript
app
    .get("/401", ({ status }) => {
        status(401)

        return "This should be 401"
    })
```

Files are also transformed to response. Simply return `Bun.file` to serve static file.
```typescript
app.get("/tako", () => Bun.file('./example/takodachi.png'))
```

To get path paramameters, prefix the path with a colon:
```typescript
app.get("/id/:id", ({ params: { id } }) => id)

// [GET] /id/123 => 123
```

To ensure the type, simply pass a generic:
```typescript
app.get<{
    params: {
        id: string
    }
}>("/id/:id", ({ params: { id } }) => id)

// [GET] /id/123 => 123
```

Wildcard works as expected:
```typescript
app.get("/wildcard/*", () => "Hi")

// [GET] /wildcard/ok => "Hi"
// [GET] /wildcard/abc/def/ghi => "Hi"
```

For a fallback page, use `default`:
```typescript
app.get("/", () => "Hi")
    .default(() => new Response("Not stonk :(", {
        status: 404
    }))

// [GET] / => "Not stonk :("
```

You can group multiple route with a prefix with `group`:
```typescript
app
    .get("/", () => "Hi")
    .group("/auth", app => {
        app
            .get("/", () => "Hi")
            .post("/sign-in", ({ body }) => body)
            .put("/sign-up", ({ body }) => body)
    })
    .listen(3000)

// [GET] /auth/sign-in => "Hi"
// [POST] /auth/sign-in => <body>
// [PUT] /auth/sign-up => <body>
```

And you can decouple the route logic to a separate plugin.
```typescript
import KingWorld, { type Plugin } from 'kingworld'

const hi: Plugin = (app) => app
    .get('/hi', () => 'Hi')

const app = new KingWorld()
    .use(hi)
    .get('/', () => 'KINGWORLD')
    .listen(3000)

// [GET] / => "KINGWORLD"
// [GET] /hi => "Hi"
```

Lastly, you can specified `hostname` to `listen` if need:
```typescript
import KingWorld, { type Plugin } from 'kingworld'

const app = new KingWorld()
    .get('/', () => 'KINGWORLD')
    .listen({
        port: 3000,
        hostname: '0.0.0.0'
    })

// [GET] / => "KINGWORLD"
```

## Handler
Handler is a callback function that returns `Response`. Used in HTTP method handler.

```typescript
new KingWorld()
    .get(
        '/', 
        // This is handler
        () => "KingWorld"
    )
    .listen(3000)
```

By default, handler will accepts two parameters: `request` and `store`.
```typescript
// Simplified Handler
type Handler = (request: ParsedRequest, store: Instance['store']) => Response

const handler: Handler = (request: {
    request: Request
    query: ParsedUrlQuery
    params: Record<string, string>
    headers: Record<string, string>
    body: Promise<string | Object>
    responseHeaders: Headers
}, store: Record<any, unknown>)
```

## Handler Request
Handler's request consists of
- request [`Request`]
    - Native fetch Request
- query [`ParsedUrlQuery`]
    - Parsed Query Parameters as `Record<string, string>`
    - Default: `{}`
    - Example:
        - path: `/hi?name=fubuki&game=KingWorld`
        - query: `{ "name": "fubuki", "game": "KingWorld" }`
- params [`Record<string, string>`]
    - Path paramters as object
    - Default: `{}`
    - Example:
        - Code: `app.get("/id/:name/:game")`
        - path: `/id/kurokami/KingWorld`
        - params: `{ "name": "kurokami", "game": "KingWorld" }`
- headers [`Record<string, string>`]
    - Function which returns request's headers
- body [`Promise<string | Object>`]
    - Function which returns request's body
    - By default will return either `string` or `Object`
        - Will return Object if request's header contains `Content-Type: application/json`, and is deserializable
        - Otherwise, will return string
- responseHeaders [`Header`]
    - Mutable object reference, will attached to response's header
    - For example, adding `CORS` to response as a plugin
- status [`(statusCode: number) => void`]
    - Function to set response status code explictly

## Store
Store is a singleton store of the application.

Is recommended for local state, reference of database connection, and other things that need to be available to be used with handler.

Store value if of 2 types:
- State: Assigned once at creation
- Ref: Assign at every request

```typescript
new KingWorld()
    .state('build', Math.random())
    .ref('random', () => Math.random())
    .get("/build", ({}, { build }) => build)
    .get("/random", ({}, { random }) => random)
    .listen(3000)

// [GET] /build => 0.5
// [GET] /build => 0.5 // Will have the same value as first request
// [GET] /date => 0.374
// [GET] /date => 0.785
// [GET] /date => 0.651
```

State will have any value assigned, eg. Function will be a function reference.
However for ref, if a value is a function, it will be called once.

This is for convenient usage of complex logic assigning at the beginning of every request.

You can assign a function to ref by assigning another callback, however if you want to assign function, please use `state` instead because function should be static.

```typescript
// ❌ Function is assigned on every request
new KingWorld()
    .ref('getRandom', () => () => Math.random())
    .get("/random", ({}, { getRandom }) => getRandom())

// ✅ Function is assigned once
new KingWorld()
    .state('getRandom', () => Math.random())
    .get("/random", ({}, { getRandom }) => getRandom())
```

### Typed Store
KingWorld accepts generic to type a store globally.

```typescript
new KingWorld<{
    store: {
        build: number
        random: number
    }
}>()
    .state('build', Math.random())
    .ref('random', () => Math.random())
    .get("/build", ({}, { build }) => build)
    .get("/random", ({}, { random }) => random)
    .listen(3000)
```

## Lifecycle
KingWorld request's lifecycle can be illustrate as the following:

```
Request -> onRequest -> route -> transform -> preHandler -> Response
```

The callback that assigned to lifecycle is called **hook**.

#### Pre Handler
- onRequest 
    - Call on new request

#### Internal
- router.find (route)
    - Find handler assigned to route

#### Post Handler
- transform [`Handler`]
    - Called before validating request
    - Use to transform request's body, params, query before validation
- preHandler [`Handler`]
    - Handle request before executing path handler
    - If value returned, will skip to Response process

Lifecycle can be assigned with `app.<lifecycle name>()`:

For example, assigning `transform` to a request:
```typescript
app
    // ? Transform params 'id' to number if available
    .transform(({ params }) => {
        if(params.id)
            params.id = +params.id
    })
```

## Local Hook
There's 2 type of hook
- Global Hook
    - Assign to every handler
- Local Hook
    - Assigned by third parameters of `Route Handler` or `app.<method>(path, handler, localHook)`
    - Affected only scoped handler

```typescript
app
    // ? Global Hook
    .transform(({ params }) => {
        if(params.id)
            params.id = +params.id + 1
    })
    .get(
        "/id/:id/:name", 
        ({ params: { id, name } }) => `${id} ${name}`,
        // ? Local hook
        {
            transform: ({ params }) => {
                if(params.name === "白上フブキ")
                    params.name = "Shirakami Fubuki"
            }
        }
    )
    .get("/new/:id", ({ params: { id, name } }) => `${id} ${name}`)
    .listen(3000)

// [GET] /id/2/kson => "3 kson"
// [GET] /id/1/白上フブキ => "2 Shirakami Fubuki"
// [GET] /new/1/白上フブキ => "2 白上フブキ"
```

You can have multiple local hooks as well by assigning it as array:
```typescript
app
    .get(
        "/id/:id/:name", 
        ({ params: { id, name } }) => `${id} ${name}`,
        {
            transform: [
                ({ params }) => {
                    if(params.id)
                        params.id = +params.id + 1
                },
                ({ params }) => {
                    if(params.name === "白上フブキ")
                        params.name = "Shirakami Fubuki"
                }
            ]
        }
    )
    .listen(3000)

// [GET] /id/2/kson => "3 kson"
// [GET] /id/1/白上フブキ => "2 Shirakami Fubuki"
// [GET] /new/1/白上フブキ => "2 白上フブキ"
```

### PreRequestHandler
Callback assign to lifecycle before routing.

As it's handle before routing, there's no `params`, `query`.

```typescript
type PreRequestHandler = (request: Request, store: Store) => void
```

Lifecycle that assigned with `PreRequestHandler`:
- onRequest

### Handler (Event)
Callback assign to lifecycle after routing.

Accept same value as [path handler, @see Handler](#handler-request)

Lifecycle that assigned with `Handler`:
- transform
- preHandler

## Transform
Use to modify request's body, params, query before validation.

```typescript
app
    .get(
        "/gamer/:name", 
        ({ params: { name }, hi }) => hi(name),
        // ? Local hook
        {
            transform: ({ params }) => {
                if(params.name === "白上フブキ")
                    params.name = "Shirakami Fubuki"
                    
                params.hi = (name: string) => `Hi ${name}`
            }
        }
    )

// [GET] /gamer/白上フブキ => "Shirakami Fubuki"
// [GET] /gamer/Botan => "Botan"
```

You can easily modify body in transform to decouple logic into separate plugin.
```typescript
new KingWorld()
	.post<{
		body: {
			id: number
			username: string
		}
	}>(
		'/gamer',
		async ({ body }) => {
			const { username } = body

			return `Hi ${username}`
		},
		{
			transform: (request) => {
				request.body.id = +request.body.id
			}
		}
	)
	.listen(8080)
```

## Schema Validation
Please use [@kingworldjs/schema](https://github.com/saltyaom/kingworld-schema) to handle typed-strict validation of incoming request.

#### Example
```typescript
import KingWorld from 'kingworld'
import schema, { S } from '@kingworldjs/schema'

new KingWorld()
    .get('/id/:id', ({ request: { params: { id } } }) => id, {
        transform: (request, store) {
            request.params.id = +request.params.id
        },
        preHandler: schema({
            params: S.object().prop('id', S.number().minimum(1).maximum(100))
        })
    })
    .listen(3000)

// [GET] /id/2 => 2
// [GET] /id/500 => Invalid params
// [GET] /id/-3 => Invalid params
```

See [@kingworldjs/schema](https://github.com/saltyaom/kingworld-schema) for more detail about schema validation.

## PreHandler
Handle request before executing path handler.
If value is returned, the value will be the response instead and skip the path handler.

Schema validation is useful, but as it only validate the type sometime app require more complex logic than type validation.

For example: Checking value if value existed in database before executing the request.

```typescript
import KingWorld, { S } from 'kingworld'

new KingWorld()
    .post<{
        body: {
            username: string
        }
    }>('/id/:id', ({ request: { body }) => {
            const { username } = await body

            return `Hi ${username}`
        }, {
        preHandler: async ({ body }) => {
            const { username } = await body
            const user = await database.find(username)

            if(user)
                return user.profile
            else
                return Response("User doesn't exists", {
                    status: 400
                })
        }
    })
    .listen(3000)
```

## Plugin
Plugin is used to decouple logic into smaller function.

```typescript
import KingWorld, { type Plugin } from 'kingworld'

const hi: Plugin = (app) => app
    .get('/hi', () => 'Hi')

const app = new KingWorld()
    .use(hi)
    .get('/', () => 'KINGWORLD')
    .listen(3000)

// [GET] / => "KINGWORLD"
// [GET] /hi => "Hi"
```

However, plugin can also be used for assigning new `store`, and `hook` making it very useful.

To register a plugin, simply add plugin into `use`.

`use` can accept 2 parameters:
- plugin [`Plugin`]
- config [`Config?`] (Optional)


```typescript
const plugin: Plugin = (
    app, 
    // Config (2nd paramters of `use`)
    { prefix = '/fbk' } = {}
) => app
        .group(prefix, (app) => {
            app.get('/plugin', () => 'From Plugin')
        })

new KingWorld()
    .use(app, {
        prefix: '/fubuki'
    })
```

To develop plugin with type support, `Plugin` can accepts generic.

```typescript
const plugin: Plugin<
    // ? Typed Config
    {
        prefix?: string
    },
    // ? Same as KingWorld<{}>(), will extends current instance
    {
        store: {
            fromPlugin: 'From Logger'
        }
        request: {
            log: () => void
        }
    }
> = (app, { prefix = '/fbk' } = {})  => 
    app
        .state('fromPlugin', 'From Logger')
        .transform(({ responseHeaders }) => {
            request.log = () => {
                console.log('From Logger')
            }

            responseHeaders.append('X-POWERED-BY', 'KINGWORLD')
        })
        .group(prefix, (app) => {
            app.get('/plugin', () => 'From Plugin')
        })

const app = new KingWorld<{
    Store: {
        build: number
        date: number
    }
}>()
    .use(plugin)
    .get('/', ({ log }) => {
        log()

        return 'KingWorld'
    })

// [GET] /fbk/plugin => "From Plugin"
```

Since Plugin have a type declaration, all request and store will be fully type and extended from plugin.

For example:
```typescript
// Before plugin registration
new KingWorld<{
    Store: {
        build: number
        date: number
    }
}>()

// After plugin registration
new KingWorld<{
    Store: {
        build: number
        date: number
    } & {
        fromPlugin: 'From Logger'
    }
    Request: {
        log: () => void
    }
}>()
```

This will enforce type safety across codebase.

```typescript
const app = new KingWorld<{
    Store: {
        build: number
        date: number
    }
}>()
    .use(plugin)
    .get('/', ({ log }) => {
        // `log` get type declaration reference from `plugin`
        log()

        return 'KingWorld'
    })
```

### Local plugin custom type
Sometime, when you develop local plugin, type reference from main instance is need, but not available after separation.

```typescript
const plugin: Plugin = (app)  => 
    app
        .get("/user/:id", ({ db, params: { id } }) => 
            // ❌ Type Error: db is not defined or smth like that
            db.find(id)
        )

const app = new KingWorld<{
    Store: {
        database: Database
    }
}>()
    .state('db', database)
    .use(plugin)
```

That's why plugin can accept the third generic for adding temporary local type but do not extend the main instance.
```typescript
const plugin: Plugin<
    {},
    {},
    // Same as KingWorld<Instance>
    {
        store: {
            db: Database
        }
    }
> = (app)  => 
    app
        .get("/user/:id", ({ db, params: { id } }) => 
            // ✅ db is now typed
            db.find(id)
        )

const app = new KingWorld()
    .state('db', database)
    .use(plugin)
```

## Async Plugin
To create an async plugin, simply create an async function the return plugin.

```typescript
const plugin = async (): Plugin => {
    const db = await setupDatabase()

    return (app) => 
        app
            .state('db', database)
            .get("/user/:id", ({ db, params: { id } }) => 
                // ✅ db is now typed
                db.find(id)
            )
}

const app = new KingWorld()
    .state('db', database)
    .use(plugin)
```

## KingWorld Instance
KingWorld can accepts named generic to type global instance.

For example, type-strict store.

```typescript
const app = new KingWorld<{
    Store: {
        build: number
    }
}>()
    .state('build', 1)
```

KingWorld instance can accept generic of `KingWorldInstance`
```typescript
export interface KingWorldInstance<
	Store extends Record<string, any> = {},
	Request extends Record<string, any> = {}
> {
	Request?: Request
	Store: Store
}
```

## Test
KingWorld is designed to be serverless, only one simple `handle` is need to be assigned to serverless function.

This also be used to create simple test environment, by simply call `handle` function.

```typescript
import { describe, expect, it } from "bun:test"

const req = (path: string) => new Request(path)

describe('Correctness', () => {
	it('[GET] /', async () => {
		const app = new KingWorld().get('/', () => 'Hi')
		const res = await app.handle(req('/'))

		expect(await res.text()).toBe('Hi')
	})
})

```

## State of KingWorld
KingWorld is an experimental web framework based on bun.

A bleeding edge web framework focused on developer friendliness, and performance, but is not recommended for production.

As KingWorld is in 0.0.0-experimental.x, API is very unstable and will change in any point of time, at-least until 0.1.0 is release.

## License
KingWorld is MIT License
