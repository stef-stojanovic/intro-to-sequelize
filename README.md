# Sequelize

## Outline
* Connecting to a Database with Sequelize
* Defining a model with Sequelize
* Using an index file to Manage your Models

## Lesson

### Connecting to a Database with Sequelize

Let's start by installing Sequelize, and a driver for our specific database, sqlite3:

<br/>

> Terminal  

```
npm install sequelize
npm install sqlite3
```

<br/>

Next, let's create a `database.js` file, and create a connection to the database in it.

<br />

> database.js

```javascript
const Sequelize = require('sequelize')
const sequelize = new Sequelize({
  dialect: 'sqlite',
  storage: './database.sqlite'
});
```

<br />

We'll use `sequelize`'s `authenticate` method to test our database connection. All communication with the database happens **asynchronously** in Node.js. 

This means that we won't be able to check whether or not we successfully connected until the asynchronous process of connecting to the database is completed, and so `authenticate` will return a promise. 

We'll use `.then` on the promise to see whether or not we connected:

<br />

> database.js

```javascript
const Sequelize = require('sequelize')
const sequelize = new Sequelize({
  dialect: 'sqlite',
  storage: './database.sqlite'
});
sequelize.authenticate()
  .then(() => {
    console.log('Connection has been established successfully.');
  })
```

<br />

Run the `database.js` file, and see if you get the success message in the terminal.

Finally, we'll `export` `sequelize` so that we can use it throughout our app:

<br />

> database.js

```javascript
const Sequelize = require('sequelize')
const sequelize = new Sequelize({
  dialect: 'sqlite',
  storage: './database.sqlite'
});
sequelize.authenticate()
  .then(() => {
    console.log('Connection has been established successfully.');
  })
module.exports= sequelize
```

<br />

### Defining a Model with Sequelize

Now let's create a models folder and a `user.js` file inside it. We'll define our `User` model inside `user.js` using `sequelize`'s `define` method, which accepts a name as it's first argument, and an object representing the properties of the model as the second argument.

<br />

> models/user.js

```javascript
const Sequelize = require('sequelize')
const sequelize = require('../database')

const User = sequelize.define('user', {
  firstName: {
    type: Sequelize.STRING,
  },
  lastName: {
    type: Sequelize.STRING
  }
});
```

<br />



Notice that we use object keys to define our column names, and objects to represent each actual column:

```javascript
firstName: {
	type: Sequelize.STRING,
}
```

Each column object has a `type` property, which we use to define the datatype for that column in the table. Here, we make `firstName` a "string" column, but there are many, many datatypes we could choose from: <http://docs.sequelizejs.com/manual/data-types.html>

This logic to define the structure of a table (the table "schema") is written in migrations in Ruby, and then the migration must be run before the model can be used. 

Sequelize supports migrations, which are super useful if your app is already in production. 

But if your app is in development, we can use the model to run our migrations and define the table schema automatically using `sequelize`'s `.sync` method:

<br />

> models/user.js

```javascript
const Sequelize = require('sequelize')
const sequelize = require('../database')

const User = sequelize.define('user', {
  firstName: {
    type: Sequelize.STRING,
  },
  lastName: {
    type: Sequelize.STRING
  }
});

sequelize.sync()
```

<br />

If you run this file, then use DB Browser to check the sqlite file, you should find that the `user` table has been created.

There are several catches here though- `.sync` doesn't work if you update an existing table. Try adding an `email` column and you'll see what I mean.

Also, we need some place to create seed data for our application. Let's solve both of these issues by with an `index.js` file in our models folder. We'll use it to:

1. Drop the existing tables
2. Recreate the tables we need
3. Seed the database

### Using an index to Manage your Models

Before we get too ahead of ourselves, let's remove the `sequelize.sync` from the User model, and then export our `User` model so that we can use it elsewhere in our app.

<br />

> models/user.js

```javascript
const Sequelize = require('sequelize')
const sequelize = require('../database')

const User = sequelize.define('user', {
  firstName: {
    type: Sequelize.STRING,
  },
  lastName: {
    type: Sequelize.STRING
  },
  email: {
    type: Sequelize.STRING
  }
});

module.exports = User
```

<br />

Next up, we'll create `index.js`, require all the necessary modules, `drop` the table schema, *then* `sync` the table schema ***then*** create seed data:

<br />

> models/index.js

```javascript
const Sequelize = require('sequelize')
const sequelize = require('../database')
const User = require('./user.js')

sequelize.drop()
    .then( () => {
    	sequelize.sync()
            .then( () => {
            	// Create seed data here
        	})
        })
```

<br />

Now run `models/index.js` and you should see the email column appear DB Browser. Neat!

Next let's actually create some seeded users. We can use the `User` model the same way we could use most any ORM- it has a `create` method which we can use to create records in our table, and it has a `find` method, which we can use to find records from our table. A complete list of Sequelize model methods can be found here: http://docs.sequelizejs.com/manual/models-usage.html>

<br />

> models/index.js

```javascript
const Sequelize = require('sequelize')
const sequelize = require('../database')
const User = require('./user.js')

sequelize.drop()
    .then( () => {
    	sequelize.sync()
            .then( () => {
            	User.create({ firstName: "Bob", lastName: "Doe", email: "bob@example.com" })
            User.create({ firstName: "Jane", lastName: "Doe", email: "jane@example.com" })
        	})
        })
```

<br />

Finally, let's export all of our models from `models/index.js`  using an object:

<br />

> models/index.js

```javascript
const Sequelize = require('sequelize')
const sequelize = require('../database')
const User = require('./user.js')

sequelize.drop()
    .then( () => {
    	sequelize.sync()
            .then( () => {
            	User.create({ firstName: "Bob", lastName: "Doe", email: "bob@example.com" })
            User.create({ firstName: "Jane", lastName: "Doe", email: "jane@example.com" })
        	})
        })

module.exports = {
    User: User
}
```

<br />

So that we could import them like this `index.js`:

<br />

> index.js

```javascript
const models = require('./models/index.js')
console.log(models.User)
```

<br />

Or, using destructing sentax and relying on `require`'s default behavior of loading `index.js`, we could rewrite it as:

<br />

> index.js

```javascript
const { User } = require('./models')
console.log(User)
```

<br />

We're not going to go into relationships today, but if we wanted to define them, we would want to do that inside of `models/index.js` as well. It would look something like this:

<br />

> models/index.js

```javascript
const Sequelize = require('sequelize')
const sequelize = require('../database')
const User = require('./user.js')
const Fruit = require('./fruit.js')

// Relationships Here:
User.hasMany(Fruit)

sequelize.drop()
    .then( () => {
    	sequelize.sync()
            .then( () => {
            	User.create({ firstName: "Bob", lastName: "Doe", email: "bob@example.com" })
            User.create({ firstName: "Jane", lastName: "Doe", email: "jane@example.com" })
        	})
        })

module.exports = {
    User: User
}
```

<br />



## Check your Understanding

Now that we know how to define, sync, and seed models, let's practice building a full CRUD RESTful API:

1. Create a new project folder
2. Install `express` to manage HTTP routing and `sequelize` to manage your database
3. Follow the folder/file conventions of this project to setup a `Fruit` model
4. Create an `index.js` in the root of your project
5. Inside of `index.js`, require your `express` and your models, and use them together to create the following RESTful endpoints:

* `index` 
  * HTTP Method: 'GET'
  * Path: '/fruits'
  * Should: return an array of all fruit from the database
* `show` 
  - HTTP Method: 'GET'
  - Path: '/fruits/:id'
  - Should: return a single fruit whose `id` matches the id in the path
* `create` 
  - HTTP Method: 'POST'
  - Path: '/fruits'
  - Body:
    - name
    - color
  - Should: create a new fruit in the database with the name and color provided
* `update` 
  - HTTP Method: 'PATCH'
  - Path: '/fruits/:id'
  - Body:
    - name
    - color
  - Should: update one of the fruit in the database with the name and color provided
* `destroy`
  * HTTP Method: 'DELETE'
  * Path: '/fruits/:id'
  * Should: Delete fruit from the database