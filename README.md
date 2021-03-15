# Prisma ORM With PostgreSQL

![](https://i.imgur.com/m9BPjmd.jpg)

## Next-generation ORM for Node.js and TypeScript
Prisma is a server-side library that helps your app read and write data to the database in an intuitive and safe way. Prisma plays well with others and is easy to integrate into your framework of choice. Prisma simplifies database access, saves repetitive CRUD boilerplate and increases type safety.

Prisma helps app developers build faster and make fewer errors with an open source ORM for PostgreSQL, MySQL and SQLite.

It consists of the following parts:

- **Prisma Client:** Auto-generated and type-safe query builder for Node.js & TypeScript
- **Prisma Migrate:** Migration system
- **Prisma Studio:** GUI to view and edit data in your database

Prisma Client can be used in any Node.js (supported versions) or TypeScript backend application (including serverless applications and microservices). This can be a REST API, a GraphQL API, a gRPC API, or anything else that needs a database.

## Why Prisma?
The main problem with the database tools that currently exist in the Node.js and TypeScript ecosystem is that they require a major tradeoff between productivity and control.

![](https://i.imgur.com/m8VTkDL.png)

As an application developer, the mental model you have for your data is that of an object. The mental model for data in SQL on the other hand are tables.

The divide between these two different representations of data is often referred to as the object-relational impedance mismatch. The object-relational impedance mismatch also is a major reason why many developers don't like working with traditional ORMs.

As an example, consider how data is organized and relationships are handled with each approach:

- Relational databases: Data is typically normalized (flat) and uses foreign keys to link across entities. The entities then need to be JOINed to manifest the actual relationships.
- Object-oriented: Objects can be deeply nested structures where you can traverse relationships simply by using dot notation.
This alludes to one of the major pitfalls with ORMs: While they make it seem that you can simply traverse relationships using familiar dot notation, under the hood the ORM generates SQL JOINs which are expensive and have the potential to drastically slow down your application (one symptom of this is the n+1 problem).

The appeal of ORMs is the premise of abstracting away the relational model and thinking about your data purely in terms of objects. While the premise is great, it's based on the wrong assumption that relational data can easily be mapped to objects which leads to lots of complications and pitfalls.

Considering the tradeoff between productivity and control again, this is how Prisma fits in:

![](https://i.imgur.com/2JtwqGm.png)

Prisma's main goal is to make application developers `more productive` when working with databases. Here are a few examples of how Prisma achieves this:

- Thinking in objects instead of mapping relational data
- Queries not classes to avoid complex model objects
- Single source of truth for database and application models
- Healthy constraints that prevent common pitfalls and antipatterns
- An abstraction that make the right thing easy ("pit of success")
- Type-safe database queries that can be validated at compile time
- Less boilerplate so developers can focus on the important parts of their app
- Auto-completion in code editors instead of needing to look up documentation

___
## Prisma vs. Sequelize as an ORM with PostgreSQL
Let's breakdown some of the comparisons between Prisma and Sequelize, the current most popular ORM for relational databases. 

### Sequelize
Sequelize is an established, mature, promise-based Node.js ORM that supports Postgres, MySQL, MariaDB, SQLite, and Microsoft SQL Server. It follows the traditional ORM pattern of defining models by extending a Model class. Operations like SELECT and INSERT are then performed using class methods. Relations are also defined using class methods like hasMany() and belongsTo().

#### Usage example
Relational query (all posts by a specific user, eager loading):

```js
const user = await User.findOne({
  where: {
    email: 'alice@sequelize.org',
  },
  include: Post,
})
```

#### Notable Features
- Familiar ORM interface and ActiveRecord usage patterns
- Detailed control over transactions and how they are executed
- Supports many databases
- Ability to use multiple read replicas
- Eager and Lazy loading of relations
- Synchronizing database based on defined Models

#### TypeScript Integration
Type definitions: Built-in
Record creation: Not Type-safe
Record fetching: Not Type-safe

Sequelize is an established, stable ActiveRecord ORM and due to its popularity and heavy use over the years, you can expect to find support from places like StackOverflow, Reddit, and GitHub Issues. However, the project has stagnated more recently and does not seem to be as active as it once was.

___
### Prisma
Prisma differs from most ORMs in that models are not defined in classes but in the Prisma schema, the main configuration and data model definition file used by the Prisma toolkit. 

In the Prisma schema you define your data source, like a PostgreSQL database, and models, like users and posts and the relations between them. Using this schema, Prisma generates a Client that exposes a Create-Read-Update-Delete (CRUD) API, which you then use to query your database. This Prisma Client functions as a rich query builder that you can use in your Node.js app to return plain JavaScript objects, not instances of a model class.

#### Usage example
Relational query (fetch all posts by a given user, given user’s email):
```js
const postsByUser = await prisma.user.findOne({ where: { email: 'alice@prisma.io' } }).posts()
```

#### Notable Features:
- Work directly with JavaScript objects and not classes and instances
- A "single source of truth," the Prisma Schema, to reduce object-relational impedance mismatch
- Built-in, type-safe database queries
- Autogenerated migrations (preview)
- Intuitive relations API
- VSCode plugin
- Autocompletion support

#### TypeScript Integration
- Type definitions: Built-in
- Record creation: Type-safe
- Record fetching: Type-safe

Although Prisma is a newer database tool and has gone through several iterations and redesigns, its unique, schema-centric architecture stands in contrast to typical ORMs which use JavaScript Classes to define models. It benefits from the backing of a funded company and paid developers, as well as an active support community and quick development cycle. It is a popular, fast-growing choice, and is here to stay.

___
## How does Prisma work?
### The Prisma schema
Every project that uses a tool from the Prisma toolkit starts with a Prisma schema file. The Prisma schema allows developers to define their application models in an intuitive data modeling language. It also contains the connection to a database and defines a generator:

```js
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}
generator client {
  provider = "prisma-client-js"
}
model Post {
  id        Int     @id @default(autoincrement())
  title     String
  content   String?
  published Boolean @default(false)
  author    User?   @relation(fields:  [authorId], references: [id])
  authorId  Int?
}
model User {
  id    Int     @id @default(autoincrement())
  email String  @unique
  name  String?
  posts Post[]
}
```

Note: The Prisma schema has powerful data modeling features. For example, it allows you to define "Prisma-level" relation fields which will make it easier to work with relations in the Prisma Client API. In the case above, the posts field on User is defined only on "Prisma-level", meaning it does not manifest as a foreign key in the underlying database.

In this schema, you configure three things:

- **Data source:** Specifies your database connection (via an environment variable)
- **Generator:** Indicates that you want to generate Prisma Client
- **Data model:** Defines your application models

### The Prisma data model
The data model is a **_collection of models_**. A model has two major functions:
#### Functions of Prisma models
- Represent a table in the underlying database
- Provide the foundation for the queries in the Prisma Client API

#### Getting a data model
There are two major workflows for "getting" a data model into your Prisma schema:

- Manually writing the data model and mapping it to the database with Prisma Migrate
- Generating the data model by introspecting a database

Once the data model is defined, you can generate `Prisma Client` which will expose CRUD and more queries for the defined models. If you're using TypeScript, you'll get full type-safety for all queries (even when only retrieving the subsets of a model's fields).

___
### Accessing your database with Prisma Client
#### Generating Prisma Client
The first step when using Prisma Client is installing the @prisma/client npm package:
```
npm install @prisma/client
```

Installing the @prisma/client package invokes the prisma generate command, which reads your Prisma schema and generates the Prisma Client code. The code will be located in node_modules/@prisma/client.

After you change your data model, you'll need to manually re-generate Prisma Client to ensure the code inside node_modules/@prisma/client get updated:
```
prisma generate
```
Notice that the @prisma/client node module references a folder named .prisma\client. The .prisma\client folder contains your generated Prisma client, and is modified each time you change the schema and run the following command.

### Using Prisma Client to send queries to your database
Once Prisma Client has been generated, you can import in your code and send queries to your database. This is what the setup code looks like.

#### Import and instantiate Prisma Client
```js
import { PrismaClient } from '@prisma/client'
const prisma = new PrismaClient()
```

Now you can start sending queries via the generated Prisma Client API, here are a few sample queries. Note that all Prisma Client queries return plain old JavaScript objects.

Learn more about the available operations in the Prisma Client API reference.

#### Retrieve all User records from the database
```js
// Run inside `async` function
const allUsers = await prisma.user.findMany()
```
#### Include the posts relation on each returned User object
```js
// Run inside `async` function
const allUsers = await prisma.user.findMany({
  include: { posts: true },
})
```
### Filter all Post records that contain "prisma"
```js
// Run inside `async` function
const filteredPosts = await prisma.post.findMany({
  where: {
    OR: [{ title: { contains: 'prisma' } }, { content: { contains: 'prisma' } }],
  },
})
```
#### Create a new User and a new Post record in the same query
```js
// Run inside `async` function
const user = await prisma.user.create({
  data: {
    name: 'Alice',
    email: 'alice@prisma.io',
    posts: {
      create: { title: 'Join us for Prisma Day 2020' },
    },
  },
})
```
#### Update an existing Post record
```js
// Run inside `async` function
const post = await prisma.post.update({
  where: { id: 42 },
  data: { published: true },
})
```

### Usage with TypeScript
Note that when using TypeScript, the result of this query will be statically typed so that you can't accidentally access a property that doesn't exist (and any typos are caught at compile-time). Learn more about leveraging Prisma Client's generated types on the Advanced usage of generated types page in the docs

___
## Typical Prisma workflows
As mentioned above, there are two ways for "getting" your data model into the Prisma schema. Depending on which approach you choose, your main Prisma workflow might look different.

### Prisma Migrate
With Prisma Migrate, Prisma's integrated database migration tool, the workflow looks as follows:

1. Manually adjust your Prisma data model
2. Migrate your development database using the prisma migrate dev CLI command
3. Use Prisma Client in your application code to access your database

#### Typical workflow with Prisma Migrate

![](https://www.prisma.io/docs/static/153657b52bde1b006c94234b5753d495/a6d36/prisma-migrate-development-workflow.png)

To learn more about the Prisma Migrate workflow, please refer the flows guide.

### SQL migrations and introspection
If for some reasons, you can not or do not want to use Prisma Migrate, you can still use introspection to pull changes update your Prisma schema from your database schema. The typical workflow when using SQL migrations and introspection is slightly different:

1. Manually adjust your database schema using SQL or a third-party migration tool
2. (Re-)introspect your database
3. Optionally (re-)configure your Prisma Client API)
4. (Re-)generate Prisma Client
5. Use Prisma Client in your application code to access your database

![](https://www.prisma.io/docs/static/dfccce02b7903dfcce8ea0605bb421b6/a6d36/prisma-introspection-development-workflow.png)
