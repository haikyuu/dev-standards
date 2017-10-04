# \[MO\] Kick start a JS GraphQL 3 layers API with Apollo-server, Dataloader and Knex \(~61min\) / 4 hours max !

## Owner: [Thomas Pucci](https://github.com/tpucci)

## Prerequisites \(~0min\)

* Have a browser installed.

## Thanks

Thanks to [Thomas Pucci](https://github.com/tpucci) for his original work on this.

## Context

During this standard, we will create a TODO ListgraphQL API.  
We will have a Todo model with a User model. We will add one example of association.

Our API will be lightly protected and use batching to minimise DB round-trips.

On top of that, you won't need **to install anything on your computer** thanks to the awesome [play-with-docker ](http://play-with-docker.com)

## Steps \(~61min\) 

> **Note**: You should commit between each step.

### Initialise a new project inside a docker container \(~6min\)



* Go to [play-with-docker](https://www.gitbook.com/book/haikyuu/todo-list-graphql-server/edit#) and create a new docker instance

![](/assets/pwd1.PNG)

* Create and go to a new directory for the project: `mkdir graphql_tuto && cd graphql_tuto`
* Init a git repository: `git init`
* Create two services with Docker-compose, one postgres database and one node server:

  * > For this step, notice that our final folder architecture looks like this:

    > ```
    > 📂 graphql_tuto
    > ├ 📂 api
    > │ └ 🗋 Dockerfile
    > ├ 📂 db
    > │ └ 🗋 Dockerfile
    > ├ 🗋 config.env
    > └ 🗋 docker-copose.yml
    > ```
  * `mkdir api db && touch .gitignore docker-compose.yml config.env api/Dockerfile db/Dockerfile`

  * In `api/Dockerfile` file, write all the commands to assemble the API image:

  ```Dockerfile
  FROM node:8.1.0-alpine

  WORKDIR /usr/src/api

  EXPOSE 3000
  CMD yarn && yarn serve
  ```

  * In a new `db/Dockerfile` file, write all the commands to assemble the db image:

  ```Dockerfile
  FROM postgres:9.6.3
  ```

  * In a new `docker-compose.yml` file, declare the two services:

  ```yml
  version: '3'
  services:
    api:
      build:
        context: ./api
      image: todolist-api
      env_file: config.env
      volumes:
        - ./api:/usr/src/api
      ports:
        - 3000:3000
      links:
        - db:db
    db:
      build:
        context: ./db
      env_file: config.env
      image: todolist-db
      ports:
        - 5431:5432
  ```

  * In a new `config.env` file, declare your environnement variable for these Docker containers:

  ```
  POSTGRES_USER=user
  POSTGRES_PASSWORD=password
  POSTGRES_DB=todolistdb
  PGDATA=/data
  DB_HOST=db
  ```

* Build these services with the command: `docker-compose build`

> **CHECK 1**: Your terminal should prompt successively these lines confirming Docker images have been built:
>
> `Successfully built 33b13ed6b80a`
>
> `Successfully tagged todollist-db:latest`
>
> ...
>
> `Successfully built 1c76c90cd825`
>
> `Successfully tagged todolist-api:latest`

Don't forget to commit

### Install nodemon and run our project \(~5min\)

* Add this to the project .gitignore: `echo "node_modules" > .gitignore`
* In the `api` folder, interactively create a `api/package.json `with the following:

```json
{
  "name": "app",
  "version": "1.0.0",
  "main": "index.js",
  "license": "MIT",
  "devDependencies": {
    "babel-cli": "^6.26.0",
    "babel-plugin-transform-class-properties": "^6.24.1",
    "babel-preset-es2015": "^6.24.1",
    "babel-preset-flow": "^6.23.0",
    "nodemon": "^1.12.1"
  },
  "scripts": {
	"serve": "nodemon index.js --exec babel-node"
	}
}
```



* Add `nodemon`, `babel-cli`, `babel-plugin-transform-class-properties`, `babel-preset-flow` and `babel-preset-es2015` to our dev dependencies: `yarn add nodemon babel-cli babel-plugin-transform-class-properties babel-preset-es2015 babel-preset-flow -D`
* In a new `api/.babelrc` file, write the babel configuration:
  ```json
  {
  "presets":[
    "es2015",
    "flow"
  ],
  "plugins":[
    "transform-class-properties"
  ]
  }
  ```
* In our `api/package.json`, write the command to launch the server:
  ```json
  "scripts": {
  "serve": "nodemon index.js --exec babel-node"
  }
  ```
* Create a new empty file ` touch api/index.js`
* Run the project: `docker-compose up` 

> **CHECK 1**: You terminal should prompt the logs of the two containers together with two different colors
>
> **CHECK 2**: From another terminal, you can access the API and see the following folder structure: `docker-compose exec api /bin/sh` then inside the container: `ls -lath`;
>
> ```
> drwxrwxr-x    3 node     node        4.0K Aug 17 12:37 .
> -rw-rw-r--    1 node     node           0 Aug 17 12:37 index.js
> drwxrwxr-x  222 node     node       12.0K Aug 17 12:37 node_modules
> -rw-rw-r--    1 node     node         426 Aug 17 12:37 package.json
> -rw-rw-r--    1 node     node       66.2K Aug 17 12:37 yarn.lock
> -rw-rw-r--    1 node     node          86 Aug 17 12:32 Dockerfile
> drwxr-xr-x    3 root     root        4.0K Aug  3 11:50 ..
> ```
>
> Exit with: `CTRL-D`
>
> **CHECK 3**: You can access the db and prompt the PostgreSQL version: `docker-compose exec db psql -U user-d todolistdb`then inside the container: `select version();`
>
> ```bash
> PostgreSQL 9.6.3 on x86_64-pc-linux-gnu, compiled by gcc (Debian 4.9.2-10) 4.9.2, 64-bit
> ```
>
> Exit with: `CTRL-D`

### Create a koa server \(~3min\)

* go into our api container: `docker-compose exec api sh`
* Install express  in our API: `yarn add express`
* In the `index.js` file, create our server:

```js
import express from "express"

const app = express()

app.get('/', function (req, res) {
  res.send('Hello World!')
})

app.listen(3000, function () {
  console.log('Server is up and running')
})
```

> **CHECK 1**: In your terminal which run docker-compose, you should see `Server is up and running`
>
> **CHECK 2**: Hitting `localhost:3000` should return `Hello World!`: `curl localhost:3000`

### Create a presentation layer with graphQL \(~6min\)

> This layer will let our API know how to present data: what data one user can query? How should front end query this data \(fields, root queries, sub queries...\)?

* go into our api container: `docker-compose exec api sh`
* Install graphQL, graphQL Server Express, graphQL tools : `yarn add graphql graphql-server-express graphql-tools body-parser`

* In a new folder `api/presentation` add a new `schema.js` file describing a simple graphQL schema:

```js
import { makeExecutableSchema } from 'graphql-tools';

const typeDefs = [`
  enum Priority {
    LOW
    MEDIUM
    HIGH
  }
  type Todo {
    id: Int!
    text: String!
    priority: Priority
    dueDate: Int #timestamp
  }

  type Query {
    todos: [Todo]
  }

  schema {
    query: Query
  }
`];

const resolvers = {
  Query: {
    todos: () => ([
      {
        id: 1,
        text: "rock the assignment",
        priority: "HIGH"
      },
      {
        id: 2,
        text: "rock the interview",
        priority: "HIGH"
      }
    ]),
  },
}

const schema = makeExecutableSchema({ typeDefs, resolvers });

export default schema;
```

* In the `api/index.js` file, add our `api` endpoint:

```js
import express from "express"
import bodyParser from 'body-parser';
import { graphqlExpress, graphiqlExpress } from 'apollo-server-express';
import schema from './presentation/schema';

const PORT = 3000;

const app = express()

app.get('/', function (req, res) {
	res.send('Hello World!')
})

app.use('/graphql', bodyParser.json(), graphqlExpress({ schema, context: {}, debug: true }));

app.listen(PORT, function () {
	console.log('Example app listening on port 3000!')
})
```

> CHECK 1: create a new directory and a file where we'll put our graphql queries: 
>
> `mkdir queries && vim queries/getTodos.json`  and copy the following into it
>
> ```json
> {
>   "query": "{todos{ text priority dueDate}}"
> }
> ```
>
> Now run the query using:
>
> ` curl --data @queries/getTodos.json localhost:3000/graphql --header "Content-Type:application/json" | jq`



### Create a business layer \(~5min\)

> This layer will contain all business logic: access controll, scoping / whitelisting, batching and caching and computed properties. More explanations can be found [here, in the bam-api repo](https://github.com/bamlab/bam-api). In this MO, we will only cover access control logic and batching / caching.

* In a new `api/business` folder add a new `todo.js `file describing our class for this business object:

```js
        //@flow
        type Priority = "HIGH" | "MEDIUM" | "LOW"
        const mockedTodos = [
            {
                id: 1,
                text: "rock the assignment",
                priority: "HIGH"
            },
            {
                id: 2,
                text: "rock the interview",
                priority: "HIGH"
            }
        ]
        
        class Hero {
            id: number;
            text: string;
            priority?: Priority;
            dueDate?: number;
        
            constructor(data) {
                this.id = data.id;
                this.text = data.text;
                this.priority = data.priority;
                this.dueDate = data.dueDate
            }
        
            static async load(ctx, args) {
                const data = mockedTodos[args.id];
                if (!data) return null;
        
                return new Todo(data);
            }
        
            static async loadAll(ctx, args) {
                const data = mockedTodos;
        
                return data.map(row => new Todo(row));
            }
        
        }
        
        export default Todo;
```

* In our previous `presentation/schema.js` file, modify our mocked resolvers to use our business layer:

```diff
+import Todo from '../business/todo';

  type Query {
    todos: [Todo]
+   todo(id: Int!): Todo
  }

Query: {
-    todos: () => ([
-      {
-        id: 1,
-        text: "rock the assignment",
-        priority: "HIGH"
-      },
-      {
-        id: 2,
-        text: "rock the interview",
-        priority: "HIGH"
-      }
-    ]),
+    todos async (_, args, ctx) => Todo.loadAll(ctx, args),
+    todo: async (_, args, ctx) => Todo.load(ctx, args),

  }

}
```

> **CHECK 1**: run the getTodos.json query: 
>
> ```
> curl --data @queries/getTodos.json localhost:3000/graphql --header "Content-Type:application/json" | jq
> ```



> ...should return our two todos
>
> **CHECK 2**: create a new query to get a todo by index:
>
> ` vim queries/getTodoByIndex.json`
>
> and fill it with 
>
> ```
> {
>   "query": "{todo(id: 0){id text priority}}"
> }
> ```
>
> ...should return our todo with its `id: 1`.

### Seed our database \(~8min\)

* Install `knex` and `pg` at the root of the project: `cd .. && yarn add knex pg`
* At the root of our project, add a `knexfile.js` file:

```js
module.exports = {
  development: {
    client: 'pg',
    connection: {
      host: 'localhost',
      port: 5431,
      user: 'heroesuser',
      password: 'heroespassword',
      database: 'heroesdb',
    },
    migrations: {
      directory: './api/db/migrations',
    },
    seeds: {
      directory: './api/db/seeds',
    },
  },
};
```

* Create a migration file: `yarn knex migrate:make add_heroes_table` and complete the new created file with this:

```js
exports.up = function(knex, Promise) {
  return knex.schema.createTableIfNotExists('Heroes', function(table) {
    table.increments('id');
    table.string('firstName');
    table.string('lastName');
    table.string('heroName');
  });
};

exports.down = function(knex, Promise) {
  return knex.schema.dropTableIfExists('Heroes');
};
```

* Create a seed file: `yarn knex seed:make heroes` and complete the new created file with this:

```js
exports.seed = function(knex, Promise) {
  return knex('Heroes').del()
    .then(function () {
      return knex('Heroes').insert([
        {id: 1, firstName: 'Clark', lastName: 'Kent', heroName: 'Superman'},
        {id: 2, firstName: 'Bruce', lastName: 'Wayne', heroName: 'Batman'},
        {id: 3, firstName: 'Peter', lastName: 'Parker', heroName: 'Spiderman'},
        {id: 4, firstName: 'Susan', lastName: 'Storm-Richards', heroName: 'Invisible Woman'},
      ]);
    });
};
```

* Run the migration and the seed: `yarn knex migrate:latest && yarn knex seed:run`

> **CHECK 1**: You can access the db and prompt content of the `Heroes` table: `docker-compose exec db psql -U heroesuser -d heroesdb` then inside the container: `select * from "Heroes";`;
>
> ```bash
>  id | firstName |    lastName    |    heroName     
> ----+-----------+----------------+-----------------
>   1 | Clark     | Kent           | Superman        
>   2 | Bruce     | Wayne          | Batman          
>   3 | Peter     | Parker         | Spiderman       
>   4 | Susan     | Storm-Richards | Invisible Woman 
> (4 rows)
> ```
>
> Exit with: `CTRL-D`

### Create a db layer with knex \(~6min\)

> This layer let our API query the data using knex query builder.

* Install `knex` and `pg` in our API: `cd api && yarn add knex pg`
* In the `api/db` folder add a new `index.js` file:

```js
import knex from 'knex';

export default knex({
  client: 'pg',
  connection: {
    host: process.env.DB_HOST,
    user: process.env.POSTGRES_USER,
    password: process.env.POSTGRES_PASSWORD,
    database: process.env.POSTGRES_DB,
  },
  debug: true,
});
```

* In a new `api/db/queryBuilders` subfolder, create a new `hero.js` file and add these few methods to query our data:

```js
// @flow
import db from '..';

class Hero {
  static async getById(id: number) {
    return db
    .first()
    .table('Heroes')
    .where('id', id);
  }

  static async getByIds(ids: Array<number>) {
    return db
    .select()
    .table('Heroes')
    .whereIn('id', ids);
  }

  static async getAll() {
    return db
    .select()
    .table('Heroes');
  }
}

export default Hero;
```

* Modify the `api/db/queryBuilders/hero.js` file in our business layer this way:

```diff
-const heroes = [
-  {
-    id: 0,
-    firstName: 'Clark',
-    lastName: 'Kent',
-  },
-  {
-    id: 1,
-    firstName: 'Bruce',
-    lastName: 'Wayne',
-  }
-];
+import HeroDB from '../db/queryBuilders/hero';

 class Hero {

   static async load(ctx, args) {
-    const data = heroes[args.id];
+    const data = await HeroDB.getById(args.id);
     if (!data) return null;

     return new Hero(data);
   }

   static async loadAll({ authToken, dataLoaders }) {
-    const data = heroes;
+    const data = await HeroDB.getAll();

     return data.map(row => new Hero(row));
   }
```

> **CHECK 1**: Using graphiql interface with the following query:
>
> ```
> {
>   hero(id:1) {
>     id
>     firstName
>     lastName
>   }
> }
> ```
>
> ...should return Clark Kent with its `id: 1`.
>
> **CHECK 2**: Using graphiql interface with the following query:
>
> ```
> {
>   heroes {
>     id
>     firstName
>     lastName
>   }
> }
> ```
>
> ...should return all 4 heroes of our database.

### Add association to our API \(~6min\)

> Association are made both in our db and in our API, in our presentation layer.

* Create a new migration: `cd .. && yarn knex migrate:make add_heroes_enemies`
* Complete the newly created migration file with this:

```js
exports.up = function(knex, Promise) {
  return knex.schema.table('Heroes', function(table) {
    table.integer('enemyId').references('id').inTable('Heroes');
  });
};

exports.down = function(knex, Promise) {
  return knex.schema.table('Heroes', function(table) {
    table.dropColumn('heroName');
  });
};
```

* Modify our `api/db/seeds/heroes.js` seeds:

```js
exports.seed = function(knex, Promise) {
  return knex('Heroes').del()
    .then(function () {
      return knex('Heroes').insert([
        {id: 1, firstName: 'Clark', lastName: 'Kent', heroName: 'Superman', enemyId: 2},
        {id: 2, firstName: 'Bruce', lastName: 'Wayne', heroName: 'Batman', enemyId: 1},
        {id: 3, firstName: 'Peter', lastName: 'Parker', heroName: 'Spiderman'},
        {id: 4, firstName: 'Susan', lastName: 'Storm-Richards', heroName: 'Invisible Woman'},
      ]);
    });
};
```

* Run these migrations: `yarn knex migrate:latest && yarn knex seed:run`
* In our business layer, modify `api/business/hero.js` this way:

```diff
class Hero {
  id: number;
  firstName: string;
  lastName: string;
+  heroName: string;
+  enemyId: number;

  constructor(data) {
    this.id = data.id;
    this.firstName = data.firstName;
    this.lastName = data.lastName;
+    this.heroName = data.heroName;
+    this.enemyId = data.enemyId;
  }
```

* In our API, in our presentation layer, modify our `api/presentation/schema.js`:

```diff
const typeDefs = [`
  type Hero {
    id: Int
    firstName: String
    lastName: String
+    heroName: String
+    enemy: Hero
  }
...
`];

const resolvers = {
  Query: {
...
  },
+  Hero: {
+    enemy: async (hero, args, ctx) => Hero.load(ctx, {id: hero.enemyId}),
+  },
}
```

> **CHECK 1**: Using graphiql interface with the following query:
>
> ```
> {
>   hero(id:1) {
>     id
>     firstName
>     lastName
>     heroName
>     enemy {
>       heroName
>     }
>   }
> }
> ```
>
> ...should return Clark Kent with its heroName and its enemy: Batman.

### Push your API to the next level: use caching with Dataloader \(~6min\)

> Trying to query heroes and their enemies'heroName will show up a N+1 problem. Indeed, our API make 5 round-trips to our database! Try yourself:
>
> ```json
> {
>     "query": "{heroes { id firstName lastName heroName enemy { heroName } }}"
> }
> ```
>
> We can reduce these calls adding caching to our business layer

* Install `Dataloader`: `cd api && yarn add dataloader`
* Add a `getLoaders` method to our `api/business/hero.js` file in our business layer:

```js
import DataLoader from 'dataloader';


class Hero {
  //...

  static getLoaders() {
    const getById = new DataLoader(ids => HeroDB.getByIds(ids));
    const primeLoaders = (heroes) => {
      heroes.forEach(hero =>
        getById.clear(hero.id).prime(hero.id, hero))
      ;
    };
    return { getById, primeLoaders };
  }
  //...
}
```

* In our `api/index.js` file, add a new dataloader to our context for each query on `/api` route:

```diff
+import Hero from './business/hero';

router.post(
  '/api',
  graphqlKoa(async ctx => {
    return {
      schema: schema,
+      context: {
+        dataLoaders: {
+          hero: Hero.getLoaders(),
+        }
+      },
      debug: true,
    };
  })
);
```

* Back in our `api/business/hero.js` business layer file, modify `load` and `loadAll` methods to use our dataloader:

```diff
  static async load(ctx, args) {
+    const data = await ctx.dataLoaders.hero.getById.load(args.id);
    if (!data) return null;

    return new Hero(data);
  }

  static async loadAll(ctx, args) {
    const data = await HeroDB.getAll();
+    ctx.dataLoaders.hero.primeLoaders(data);

    return data.map(row => new Hero(row));
  }
```

* Protect `loader.load()` function call if no argument is supplied:

```diff
  static async load(ctx, args) {
+    if (!args.id) return null;
    const data = await ctx.dataLoaders.hero.getById.load(args.id);
    if (!data) return null;

    return new Hero(data);
  }
```

> **CHECK 1**: Using graphiql interface with the following query:
>
> ```
> {
>   heroes {
>     id
>     firstName
>     lastName
>     heroName
>     enemy {
>       heroName
>     }
>   }
> }
> ```
>
> ...should return all heroes and their enemies and your terminal should prompt only one request to the DB.
>
> **CHECK 2**: Using graphiql interface with the following query:
>
> ```
> {
>   h1: hero(id:1) {
>     id
>     firstName
>     lastName
>     heroName
>     enemy {
>       heroName
>     }
>   }
>   h2: hero(id:2) {
>     id
>     firstName
>     lastName
>     heroName
>     enemy {
>       heroName
>     }
>   }
> }
> ```
>
> ...should return Clark Kent and Bruce Wayne; and only one _SELECT_ call should have beeen made to our DB.

### Add access control to our API \(~5min\)

> This is a very simple example, for a more advanced solution, prefer using [Koa Jwt](https://github.com/koajs/jwt).

* In a new `api/utils.js` file, add these two methods to parse Authorization header and verify token:

```js
export const parseAuthorizationHeader = (req) => {
  const header = req.headers.authorization;

  if (typeof header === 'undefined' || header === 'null') {
    return null;
  }

  const [, scheme, token] = (/(\w+) ([\w.-]+)/g).exec(header);

  return token;
};

// Not production-ready: this is a simple example for the tutorial
export const verifyToken = token => new Promise((resolve, reject) => {
  if (token !== 'authorized') {
    const error = new Error('UNAUTHORIZED');
    error.code = 401;
    reject(error);
  }
  resolve();
});
```

* In our `api/index.js` file, parse authorization header and pass it to our context:

```diff
+import { parseAuthorizationHeader } from './utils';

router.post(
  '/api',
  graphqlKoa(async ctx => {
    return {
      schema: schema,
      context: {
+        authToken: parseAuthorizationHeader(ctx.req),
        dataLoaders: {
          hero: Hero.getLoaders(),
        }
      },
      debug: true,
    };
  })
);
```

* In our business layer, modify `api/business/hero.js`:

```diff
+import { verifyToken } from '../utils';


  static async load(ctx, args) {
+    await verifyToken(ctx.authToken);

  static async loadAll(ctx, args) {
+    await verifyToken(ctx.authToken);
```

> **CHECK 1**: In **Postman**, making a _POST_ request to `localhost:3000/api` which content-type is _JSON\(application/json\)_ with the following raw body:
>
> ```json
> {
>     "query": "{hero(id:1) { id firstName lastName heroName }}"
> }
> ```
>
> ...should return `UNAUTHORIZED`.
>
> **CHECK 2**: In **Postman**, making a _POST_ request to `localhost:3000/api` which content-type is _JSON\(application/json\)_ with the following raw body:
>
> ```json
> {
>     "query": "{heroes { id firstName lastName heroName }}"
> }
> ```
>
> ...should return `UNAUTHORIZED`.
>
> **CHECK 3**: In **Postman**, making a _POST_ request to `localhost:3000/api` which content-type is _JSON\(application/json\)_ and _Authorization Header_ is `Bearer authorized` with the following raw body:
>
> ```json
> {
>     "query": "{hero(id:1) { firstName lastName }}"
> }
> ```
>
> ...should return Clark Kent.
>
> ![](assets/authorization.png)

### Troubleshooting: Accessing data by id in the correct order \(~5min\)

> You should notice that in **Postman** making a _POST_ request to `localhost:3000/api` which content-type is _JSON\(application/json\)_ and _Authorization Header_ is `Bearer authorized` with the following raw body:
>
> ```json
> {
>     "query": "{h1: hero(id:1) { id firstName lastName heroName enemy { heroName } } h2: hero(id:2) { id firstName lastName heroName enemy { heroName } }}"
> }
> ```
>
> ...returns the same than the following request \(ids switched\):
>
> ```json
> {
>     "query": "{h1: hero(id:2) { id firstName lastName heroName enemy { heroName } } h2: hero(id:1) { id firstName lastName heroName enemy { heroName } }}"
> }
> ```
>
> This is due to our DB query: `select * from "Heroes" where "id" in (1, 2)` return the same result than: `select * from "Heroes" where "id" in (2, 1)`.

* In `utils.js`, add the following method:

```js
export const orderByArgIdsOrder = ids => ("array_position(string_to_array(?, ',')::integer[], id)", ids.join(','));
```

* In our db layer, modify `api/db/queryBuilders/hero.js` like this:

```diff
+import { orderByArgIdsOrder } from '../../utils';

class Hero {

  static async getByIds(ids: Array<number>): Promise<Array<CostDBType>> {
    return db
    .select()
    .table('Heroes')
+    .whereIn('id', ids)
+    .orderByRaw(orderByArgIdsOrder(ids));
  }
```

> **CHECK 1**: In **Postman**, making a _POST_ request to `localhost:3000/api` which content-type is _JSON\(application/json\)_ and _Authorization Header_ is `Bearer authorized` with the following raw body:
>
> ```json
> {
>     "query": "{h1: hero(id:2) { heroName } h2: hero(id:1) { heroName }}""
> }
> ```
>
> ...should return Batman \(as `h1`\) then Superman \(as `h2`\).

## Next steps

* Add [Koa Jwt](https://github.com/koajs/jwt)
* Add graphiQL with authorization header \(get inspired by [BAM API](https://github.com/bamlab/bam-api)\)



