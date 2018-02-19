Simple application w/ Node.js + Koa + PostgreSQL

From Koa introduction:
> Koa is a new web framework designed by the team behind Express, which aims to be a smaller, more expressive, and more robust foundation for web applications and APIs. Through leveraging generators Koa allows you to ditch callbacks and greatly increase error-handling. Koa does not bundle any middleware within core, and provides an elegant suite of methods that make writing servers fast and enjoyable.


### Requirements

Koa requires Node `7.6.0` or higher for `async/await` syntax.  
You can use older Node versions with Babel (see more about it at [Koa installation section](http://koajs.com/)).  
PostgreSQL version does not matter. We will use the latest one.


### Environment

#### Node.js

If Node is not installed yet, you can download it [here](https://nodejs.org/en/download/). If you are a lucky owner of Linux or macOS, package managers can be used - this method described [here](https://nodejs.org/en/download/package-manager/).  
To check that the Node installation is correct, you can run the command:

```bash
$ node --version
```

That will print the Node version.  
Also for NPM:

```bash
$ npm --version
```

#### Dependencies installation

Create directory where all the code will be located. Run terminal and open this directory.

Let's initialize the package environment:

```bash
$ npm init
```

NPM will ask you a few questions. Type answers for package name of author.

```bash
$ npm init

This utility will walk you through creating a package.json file.
It only covers the most common items, and tries to guess sensible defaults.

See `npm help json` for definitive documentation on these fields
and exactly what they do.

Use `npm install <pkg>` afterwards to install a package and
save it as a dependency in the package.json file.

Press ^C at any time to quit.
package name: (ex) exampleapp
version: (1.0.0)
description: Example application on Koa and PostgreSQL
entry point: (index.js)
test command:
git repository:
keywords:
author: Nariman Safiulin <woofilee@gmail.com>
license: (ISC)
About to write to <..>\package.json:

{
  "name": "exampleapp",
  "version": "1.0.0",
  "description": "Example application on Koa and PostgreSQL",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "Nariman Safiulin <woofilee@gmail.com>",
  "license": "ISC"
}


Is this ok? (yes)
```

You should see the `package.json` file that NPM has created.

Install [`koa`](https://github.com/koajs/koa):

```bash
$ npm i koa
```

Also we need something for requests routing. We will use a [`koa-router`](https://github.com/alexmingoia/koa-router) - popular and full-featured package.  
Another one - [`koa-logger`](https://github.com/koajs/logger) - for development logging.

```bash
$ npm i koa-router koa-logger
```

For database we will use a [`node-postgres`](https://github.com/brianc/node-postgres) driver. It's a simple client, not a ORM.

```bash
$ npm i pg
```

#### PostgreSQL

You can download PostgreSQL [here](https://www.postgresql.org/download/) with a portable binaries for fast start w/o installation proccess.  
To check the PostgreSQL installation, run the command:

```bash
$ sudo -u postgres psql
```

On Windows (type your own Windows user name instead of `postgres`, if you didn't create a database manually):

```
> psql -U postgres
```

On Linux and macOS a PostgreSQL server should be running yet after installation via package manager.
Otherwise, or if you have Windows, follow next commands:

```
> initdb -U postgres -D data
> pg_ctl -D data start
```

A default `postgres` database username is recommended, `data` - a database storage directory, you can provide any other path.  
After that a `pg_ctl` command will start the PostgreSQL server (provide a path to the database storage).

Check again, you should see a PostgreSQL prompt:

```sql
postgres=#
```

Type `\q` to exit.

Default database `postgres` was created for us.


### Hello, World!

Create file `app.js` and copy the next code. It's a  modified version of standart Koa example.

```js
const Koa = require('koa');
const Router = require('koa-router');
const Logger = require('koa-logger');

const app = new Koa();
const router = new Router();

// Response to the World to the GET requests
router.get('/', async (ctx) => {
  ctx.body = 'Hello, World!\n';
});

// Response by name to the GET requests, :name is URL fragment/argument
router.get('/:name', async (ctx) => {
  ctx.body = `Hello, ${ctx.params.name}!\n`;
});

// Development logging
app.use(Logger());
// Add routes and response to the OPTIONS requests
app.use(router.routes()).use(router.allowedMethods());

// Listen the port
app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

Now you can start the server:

```bash
$ node app.js
```

If the server has successfully started, open the browser and go to `http://localhost:3000/`. You should see the `Hello, World!` page. Also, try to type your name, for example, `http://localhost:3000/Nariman`.  
In your terminal you should see something like that:

```bash
$ node app.js

Server running on port 3000
  <-- GET /
  --> GET / 200 13ms 14b
  <-- GET /favicon.ico
  --> GET /favicon.ico 200 7ms 20b
  <-- GET /Nariman
  --> GET /Nariman 200 10ms 16b
  <-- GET /favicon.ico
  --> GET /favicon.ico 200 7ms 20b
```

Let's connect to the database.   
We will use the connection pool, because it's a common way to use database drivers, especially for PostgreSQL. Pool has advatages over manual connections, due to each new manual connection can take some time for connection establishment and it requires some management to control the number of simultaneous connections.

Add the dependency:

```js
const { Pool } = require('pg');
```

Right before the server booting code (before `app.listen` call):

```js
app.pool = new Pool({
  user: 'postgres',
  host: 'localhost',
  database: 'postgres',
  password: '', // Password is empty be default
  port: 5432, // Default port
});
```

Then, change our routes, to query database for `Hello` string.

```js
router.get('/', async (ctx) => {
  const { rows } = await ctx.app.pool.query('SELECT $1::text as message', ['Hello, World!'])
  ctx.body = rows[0].message;
});

router.get('/:name', async (ctx) => {
  const { rows } = await ctx.app.pool.query('SELECT $1::text as message', [`Hello, ${ctx.params.name}!`])
  ctx.body = rows[0].message;
});
```

Note that Koa can auto-translate the result into JSON, if the response is not a string. If you just return `rows`, you'll see the JSON response.

Save and restart the server.


### In conclusion

We wrote everything in one file. Of course, for a real project, you should create the project structure, and also take care of closing the PostgreSQL pool and Koa connections when shutting down the server.

You can also use [other](https://github.com/sindresorhus/awesome-nodejs#database) database libraries, like [Sequelize](http://sequelizejs.com/), [Bookshelf](http://bookshelfjs.org/), [Massive](https://github.com/dmfay/massive-js).


### References

* Node deps
    * [koa](https://github.com/koajs/koa) ([koajs.com](http://koajs.com/))
    * [koa-router](https://github.com/alexmingoia/koa-router)
    * [koa-logger](https://github.com/koajs/logger)
    * [node-postgres](https://github.com/brianc/node-postgres) ([node-postgres.com](https://node-postgres.com))
* [PostgreSQL](https://www.postgresql.org/)
* [How To Install and Use PostgreSQL on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04)
* [Building a RESTful API with Koa and Postgres](http://mherman.org/blog/2017/08/23/building-a-restful-api-with-koa-and-postgres/#.Wnr4JqiWbDc)
* [Building A Server-Side Application With Async Functions and Koa 2](https://www.smashingmagazine.com/2016/08/getting-started-koa-2-async-functions/)
* [Koa examples](https://github.com/koajs/examples)
* [Koa wiki](https://github.com/koajs/koa/wiki) - links to the other useful stuff for Koa.
* [Awesome Node.js](https://github.com/sindresorhus/awesome-nodejs)
