# REST API Review - Node/Express
## Knex, PostgreSQL, JWT, deployed on Heroku
### With an emphasis on git hygiene and deployment practices

<br />

**requirement:** you must have [pgadmin](https://www.pgadmin.org/download/) installed for this review.

<br />

This document is meant to serve as a checklist review of materials taught in the Node Unit of the Full Stack Web Curriculum at Lambda School.

Upon completion of this review, you should have a simple application that _could_ be used as a boilerplate, upon which you can build out custom models, routes, and helpers to suit individual needs. However, don't use a boilerplate to do something like this until you can build it from scratch on your own. Do the reps to earn it, this document is just a guide.

The code samples included in this review should be considered a 'bare minimum' and should be embellished on in any way. In some instances there is no code included, simply instructions to author something on your own. Seriously don't just copy this. Write excellent tests, thoughtful documentation, and return meaningful data. Some commands and code examples may differ slightly from what you were taught in lectures, but the underlying principles will remain the same.


## Initial Commit

Create a directory for your API to live in

In your terminal:

- [ ]	`mkdir < server-directory-name > && cd < server-directory-name >`

> **NOTE:** the `&&` in this first command executes both commands in sequence, or one after the other. Having this command on one line creates the directory and then changes into it so that the following commands are executed in the newly created root directory. Great for this use case, but use with caution in other contexts.
> 
> All further terminal commands in this review take place in the root directory.

- [ ] `git init`
- [ ] `npx gitignore node` //creates .gitignore for node

<br />

## Connect to remote


- [ ] Login to your github account and create a new repo. Don't add a README or a .gitignore, we're doing that ourselves. Give the repo a meaningful name (I like to use the same name as the local directory we created above), and once it's created copy the URL provided.

In your terminal:

- [ ] `git add .gitignore`
- [ ] `git commit -m "Initial Commit"`
- [ ] `git remote add origin <remote_repo_rl>` (the url from your github remote)
- [ ] `git push -u origin master`
- [ ] `git checkout -b "initialize"`
- [ ] `git push -u origin initialize`

Now you have a remote repo connected to your local repo, and you're currently developing on a newly published branch called _initialize_.


<br />


## Initalizes Node/NPM

In your terminal:

- [ ] `npm init -y` //creates package.json
- [ ] `git add package.json`
- [ ] `git commit -m "Initializes Node/NPM"`

> **NOTE:** The `-y` flag answers 'yes' to all `npm init` configuration questions. We don't need anything more configured than the stock package.json for now, but explore `npm init` without the `-y` flag later so you have a better idea of what's going on here.

<br />

## Installs dependencies

In your terminal:

- [ ] `npm i express helmet cors knex knex-cleaner dotenv pg bcryptjs jsonwebtoken`
- [ ] `npm i nodemon cross-env jest supertest -D`
- [ ] `git add package.json package-lock.json`
- [ ] `git commit -m "Installs dependencies"`

<br />


## Updates package.json

- [ ] Add server (and start) script(s):

<br />

_package.json_

```json
"scripts": {
	"server": "nodemon",  //defaults to index.js
	"start": "node index.js" (for deployments)
}
```
<br />

In your terminal:

- [ ] `git add package.json`
- [ ] `git commit -m "Updates package.json"`
- [ ] `git push`

> **NOTE:** You may notice that the `-m` message in each of my commits so far is taken from the header of that section. Throughout this review, continue to follow the commit format laid out above, and if you need to see which files to `add` use `git status`. Following this commit flow will yield a really nice commit history that can also serve as a checklist on future reps through this development process. GIT HYGIENE IS SO IMPORTANT TO _GOOD_ EMPLOYERS!!!!


<br />

## Writes test for Server endpoint

Start a new branch called `tests/server-endpoint`:

- [ ] `git checkout -b "tests/server-endpoint"`
- [ ] `git push -u origin tests/server-endpoint`

Add the following files to the root directory, and don't forget the `API` directory:

- [ ] index.js
- [ ] API/server.js
- [ ] API/server.spec.js

- [ ] build your application's entry point in index.js

_index.js_

```javascript
const server = require('./API/server.js');

const port = process.env.PORT || 5000;
server.listen(port, () => {
  console.log(`\n<<< Server running on port ${port} >>>\n`);
})
```

- [ ] and add a simple test:

_API/server.spec.js_

```javascript
const request = require("supertest");

const server = require("./server.js");

it("should set db environment to testing", function() {
  expect(process.env.DB_ENV).toBe("testing");
});

describe("server", function() {
  describe("GET /", function() {
    it("should return 200", function() {
      // run the server
      // make a GET request to /
      // see that the http code of response is 200
      return request(server)
        .get("/")
        .then(res => {
          expect(res.status).toBe(200);
        });
    });

    it("should return HTML", function() {
      return request(server)
        .get("/")
        .then(res => {
          expect(res.type).toMatch(/html/i);
        });
    });
  });
});
```

- [ ] then add a test script to our package.json:

_package.json_

```json
"scripts": {
	"server": "nodemon",
	"start": "node index.js",
	"test": "cross-env DB_ENV=testing jest --watch"
},
```

- [ ] And add a very simple express server:

_API/server.js_

```javascript
const express = require('express');

const server = express();

module.exports = server;
```

Then run our tests to watch them fail first,

in your terminal:

- [ ] `npm run test`

- [ ] Now commit this work.


<br />

## Adds server endpoint

- [ ] Create a new branch called `feat/server-endpoint` and push it up to the remote.

- [ ] Let's finish building out our basic server endpoint so that test can pass:

_API/server.js_

```javascript
const express = require('express');
const cors = require('cors');
const helmet = require('helmet');

const server = express();

server.use(helmet());
server.use(cors());
server.use(express.json());

server.get('/', (req, res) => {
  res.send('<h1>🚀</h1>');
})

module.exports = server;
```

Then, in your terminal:

- [ ] `npm run server`, and once you see the server is running in your console,
- [ ] `npm run test` (in a new terminal window, or stop the server first)

- [ ] Commit this work, as long as your test passes.

<br />

## Adds Logger middleware

Adding a logger middleware early is a good idea, so let's do that now.

- [ ] Make a new branch called `middleware/logger` and push it up to the remote.

- [ ] In your root directory, add a new directory called `middleware`
- [ ] Create a file called `logger.js` inside of `middleware`

_middleware/logger.js_

```javascript
module.exports = logger;

function logger(req, res, next) {
  console.log(`${req.method} to ${req.url} at ${new Date().toISOString()}`)
  next();
};
```

- [ ] and then let's `.use` the logger in our server.js

_API/server.js_

```javascript
const express = require('express');
const cors = require('cors');
const helmet = require('helmet');

const logger = require('../middleware/logger');

const server = express();

server.use(helmet());
server.use(cors());
server.use(express.json());
server.use(logger);

server.get('/', (req, res) => {
  res.send('<h1>🚀</h1>');
})

module.exports = server;
```

- [ ] Run your server and point your browser to `localhost:5000`. You should see the rocket emoji in your browser window, and when you come back to your terminal you should see something like: `GET to / at dateTime`, which means your logger is logging.

- [ ] Commit this work and push it up.

<br />

## Deploys application to Heroku staging

We now have a super simple express server running, so it's time to deploy! Try to deploy as soon as you have _something_ working, so that whenever you're ironing out any issues in staging, you'll only have to troubleshoot the smallest codebase possible. Deploy early, commit often!

<br />

---

To explain the 3 different environments we'll be using:

- **Development** is the environment on our local machines
	- This is where we write the code
- **Staging** is (in this context) the environment on Heroku that _continuously_ deploys anything on `master`
	- This is where we test and troubleshoot any deploy issues
- **Production** is (in this context) the environment on Heroku that we _manually_ deploy from `master` once staging looks good
	- This is where any client that needs our API accesses it (like a React app)
	- We _only_ manually deploy this branch once the new features on our staging branch are working as we expect, so that we can be confident in our production server's stability.

---
<br />

- [ ] [Signup/login](https://dashboard.heroku.com/login) to a free Heroku account.

- [ ] Once logged into your Heroku dashboard, click on `New` in the top Right corner, and then on `Create new app`

- [ ] Give your app a meaningful name, and be sure to use the word `staging` to differentiate it from the `production` app we'll build in a bit. Then click on `Create app`

- [ ] Once your app is created, navigate to the `Deploy` tab and select `GitHub` as the Deployment Method.

- [ ] Connect the app to your remote GitHub respoitory, and enable automatic deploys on `master` branch.

Now, since our `master` branch currently only contains a .gitignore file, we'll need to merge our latest `middleware/logger` branch into `master` so Heroku will have something to deploy.

<br />

In your terminal:

- [ ] `git checkout master`

since we know there are no changes on the remote that we need to pull into our branch first, we can:

- [ ] `git merge middleware/logger`
- [ ] `git push origin master`

<br />

> If there were remote changes that we needed to pull into our local branch first, we would need to instead run:
> 
> - `git checkout master`
> - `git pull --rebase origin master`
> - `git checkout middleware/logger`
> - `git pull --rebase origin master`
> - `git checkout master`
> - `git merge middleware/logger`
> - `git push origin master`

<br />

Now in your Heroku Dashboard, you can click on the `Activity` tab and watch Heroku build the application.

- [ ] Once you see a `Build Succeeded` and a `Deployed` message in the `Activity ` tab, click on 'Open app' in the top right corner and hope for a rocket!

- [ ] While you're in your Heroku dashboard, go ahead and create a new app for production following the above guidance. Connect it to the exact same repository just don't deploy it yet.

<br />


## Updates Documentation

Now that we have a staging app deployed, it's time to create and update our README.md to reflect the changes we're about to take live on our production deployment. You'll want to follow this deployment flow to allow for the smoothest introduction of new features for anyone working on a client that would be consuming this API: 
> deploy to staging > test/troubleshoot staging deploy > update docs > deploy to production

If you follow this flow every time you're deploying new features, you'll save yourself the rush to explain everything you've changed. All the new features will be documented in the `master` README before the `production` deploy completes, so your team will remain informed about the current state of the deploy.

Use the [Standard Readme](https://github.com/RichardLitt/standard-readme) spec for guidance, and include plenty of meaningful info in your README like a link to the deployed production API (if you already created your production deploy in Heroku, the URL for the production app will be `https://NAME-OF-PRODUCTION-APP-HERE.herokuapp.com`)

- [ ] Create and edit the README.md on a new branch named `docs/README` and push it up to the remote. Then pull that branch into `master`

<br />

## Deploys application to Heroku production

Now that we've updated our docs and been through any troubleshooting on the staging app, we can deploy to production.

- [ ] In your Heroku Dashboard, navigate to the Production app (**not** staging) that we created earlier, and go to the `Deploy` tab. Make sure that automatic deploys are turned off, then manually deploy the `master` branch.

Your staging app is continuously deploying from `master` because we want our newest edits to be live for us to test asap, but we manually deploy production so that we can be sure everything is working correctly before we take our new edits live.

- [ ] Once your production app is deployed, test it out like you did for staging (hopefully rocket!) and pat yourself on the back.

Next we'll create our database and add routes for users to register and login.

<br />

## Configures knex

- [ ] First, since we're working with postgres, we'll need to create a database locally. If you haven't already, download [PG Admin](https://www.pgadmin.org/download/) and create a new db for your project. I'm making a drink recipe API later on, so I'll call mine `cocktailRecipeDB`. Call yours something relevant to your application.

Back in your terminal (in the root folder of this project)

- [ ] Checkout a new branch called `config/knex` and push it to the remote
- [ ] `npx knex init`

This creates a new knexfile.js at the root of our project. Edit it to look similar to this:

```javascript
// Update with your config settings.

module.exports = {

  development: {
    client: 'pg',
    connection: 'postgres://localhost/cocktailRecipeDB',
    migrations: {
      directory: './data/migrations',
    },
    seeds: {
      directory: './data/seeds',
    },
    pool: {
      min: 2,
      max: 10,
    },
  },

  production: {
    client: 'pg',
    connection: process.env.DATABASE_URL,
    migrations: {
      directory: './data/migrations',
    },
    pool: {
      min: 2,
      max: 10
    },
  }

};

```
> **NOTE:** I'm using an unsecured Postgres DB here, it should be pretty easy to find the correct config for adding credentials.

Notice that we don't include a staging environment here. That's because we have two seperate heroku deploys and we want them to mirror each other as closely as possible, so we'll also point our heroku 'Staging' app to the 'Production' knex config. It'll be using a different db anyhow so having the one env in our simple context is ideal.

- [ ] Commit and push

<br />


## Creates migration for users table

In your terminal:

- [ ] `npx knex migrate:make users`

- [ ] which will create a `_users.js` file in `data/migrations/`, edit it to look like so:

__users.js_

```javascript
exports.up = function(knex) {
  return knex.schema.createTable('users', users => {

    users.increments();

    users
      .string('username', 128)
      .notNullable()
      .unique();

    users 
      .string('password', 128)
      .notNullable();

      users 
      .string('email', 128)
      .notNullable()
      .unique();
  });
};

exports.down = function(knex) {
  return knex.schema.dropTableIfExists('users');
}
```

- [ ] Commit and push this work

I am not covering seeding in this review. We will use insomnia or postman to populate our database.

- [ ] **OPTIONAL:** On your own, using google and knexjs.org, find information about seeding a postgres db. Watch out for those primary keys!

## Configures db entry point

- [ ] in the `data/` directory, add a file called `dbConfig.js` and configure knex to process the correct environment.

_data/dbConfig.js_


```javascript
const knex = require('knex');

const knexConfig = require('../knexfile.js');

const environment = process.env.DB_ENV || 'development'

module.exports = knex(knexConfig[environment]);
```

- [ ] Commit this work and push it up

## Adds Users Route

- [ ] Create a new branch called `feat/users-route` and publish it to the remote

Now we'll build out the model, router, and basic validation for accessing the `Users` resource, so we can use it when registering or logging in.

- [ ] At the root of our project, add a `users/` directory, and add to it 3 files:
	- [ ] user-helpers.js
	- [ ] user-model.js
	- [ ] user-router.js 

	
- [ ] First we'll write our knex query functions to look something like this:

_users/user-model.js_

```javascript
const db = require('../data/dbConfig');

module.exports = {
  add,
  find,
  findBy,
  findById,
  update,
  remove
};

function find() {
  return db('users').select('id', 'username', 'email');
}

function findBy(filter) {
  return db('users').where(filter);
}

async function add(user) {
  const [id] = await db('users').insert(user, 'id');
  return findById(id);
}

function findById(id) {
  return db('users')
    .select('id', 'username', 'email')
    .where({ id })
    .first();
}

function update(id, user) {
  return db('users')
    .where('id', Number(id))
    .update(user);
}

function remove(id) {
  return db('users')
    .where('id', Number(id))
    .del();
}
```	

- [ ] Now in our router, we're only going to write a `GET` to `/`, a `GET` to `/:id`, and `DELETE` to `/:id`. We'll take care of `ADD` in our Register endpoint later.

_users/user-router.js_

```javascript
const router = require('express').Router();

const Users = require('./user-model.js');

router.get('/', (req, res) => {
  Users.find()
    .then(users => {
      res.status(200).json(users);
    })
    .catch(err => res.send(err));
});

router.get('/:id', (req, res) => {
  const id = req.params.id;
  if (!id) {
    res.status(404).json({ message: "The user with the specified id does not exist." });
  } else {
    Users.findById(id)
    .then(user => {
      res.status(201).json(user)
    })
    .catch(err => {
      res.status(500).json({ message: 'The user information could not be retrieved.' });
    })
  }
});

router.delete('/:id', (req, res) => {
  const id = req.params.id;
  if (!id) {
    res.status(404).json({ message: "The user with the specified ID does not exist." })
  }
  Users.remove(id)
   .then(user => {
     res.json(user);
   })
    .catch(err => {
      res.status(500).json({ message: 'The user could not be removed' });
    })
});

module.exports = router;
```

- [ ] Let's navigate now into our server and `.use` our new route.

```javascript
const express = require('express');
const cors = require('cors');
const helmet = require('helmet');

const logger = require('../middleware/logger');

const usersRouter = require("../users/user-router");

const server = express();

server.use(helmet());
server.use(cors());
server.use(express.json());
server.use(logger);

server.use("/api/users", usersRouter);

server.get('/', (req, res) => {
  res.send('<h1>🚀</h1>');
})

module.exports = server;
```

- [ ] Test out your new route by starting up your server and pointing postman or insomnia toward `localhost:5000/api/users`. You should return an empty array with a status of `200`.

- [ ] Commit this work and push


## Adds validation for `POST`

Always remember to add good validation for any `POST` or `PUT` methods. The following is an incredibly basic validation function, make sure that when you're validating it's in a more meaningful way.

- [ ] in your `user-helper.js`

_users/user-helper.js_

```javascript
module.exports = {
  validateUser
};

function validateUser(user) {
  let errors = [];

  if (!user.username || user.username.length < 2) {
    errors.push('Username must contain at least 2 characters');
  }

  if (!user.password || user.password.length < 4) {
    errors.push('Password must contain at least 4 characters');
  }

  return {
    isSuccessful: errors.length > 0 ? false : true,
    errors
  };

} 
```

- [ ]Commit and push

## Restricts Users Route

- [ ] Checkout a new branch named `feat/auth` and push it to remote.

Now we're going to prepare some middleware to check for and verify a JSON web token.

- [ ] In the root directory, add another directory named `auth`
	- [ ] Add `restricion.js` to your new `auth` directory.

_auth/restriction.js_

```javascript
const jwt = require('jsonwebtoken');

module.exports = (req, res, next) => {

  const token = req.headers.authorization;

  if (token) {
    const secret = process.env.JWT_SECRET || "Let me tell you a myth about secrets..";

    jwt.verify(token, secret, (err, decodedToken) => {
      if (err) {
        res.status(401).json({ message: 'Invalid Credentials' });
      } else {
        req.decodedJwt = decodedToken;
        next();
      }
    });
  } else {
    res.status(400).json({ message: 'No credentials provided' });
  }

};
```

- [ ] Then go into your user-router and implement this middleware

_users/user-router.js_

```javascript
const restricted = require('../auth/restriction.js');

router.get('/', restricted, (req, res) => {
  Users.find()
  	...
  	...
}  	
```

- [ ] Test out this restriction by trying to `GET /api/users` in Insomnia or Postman and looking for the relevant error.

- [ ] Commit and push

## Adds tests for Login and Register endpoints

- [ ] Write meaningful tests for both Login and Register endpoints that take into account any constraints of your DB (again, since we're using Postgres here watch out for those primary keys!)

## Adds Login and Register endpoints

- [ ] Create a new branch called `feat/auth` and push it to your remote

- [ ] In `auth/` create 3 files
	- [ ] auth-helpers.js
	- [ ] login-router.js
	- [ ] register-router.js

	
- [ ] First write a helper function to get the JSON Web Token

_auth/auth-helpers.js_

```javascript
const jwt = require('jsonwebtoken');

module.exports = {
  getJwt
}

function getJwt(username) {
  const payload = {
    username
  };

  const secret = process.env.JWT_SECRET || "Let me tell you a myth about secrets..";

  const options = {
    expiresIn: '1d'
  };

  return jwt.sign(payload, secret, options)
}
``` 

- [ ] Then write a route for logging in, using our new `getJWT()` function

_auth/login-router.js_

```javascript
const router = require('express').Router();
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');

const Users = require('../users/user-model');
const Token = require('./auth-helpers.js');

router.post('/', (req, res) => {
  
  let { username, password } = req.body;

  Users.findBy({ username })
    .first()
    .then(user => {

      if (user && bcrypt.compareSync(password, user.password)) {
        
        const token = Token.getJwt(user.username);

        res.status(200).json({
          id: user.id,
          username: user.username,
          token
        });
      } else {
        res.status(401).json({ message: 'Invalid credentials' });
      }
    })
    .catch(error => {
      res.status(500).json(error);
    });
})


module.exports = router;
```

- [ ] And a route for registering, using the same getJWT function.

_auth/register-router.js_

```javascript
const router = require('express').Router();
const bcrypt = require('bcryptjs');

const Users = require('../users/user-model.js');
const Token = require('./auth-helpers.js');
const { validateUser } = require('../users/user-helpers.js');

router.post('/', (req, res) => {

  let user = req.body;

  const validateResult = validateUser(user);

  if (validateResult.isSuccessful === true) {
    
    const hash = bcrypt.hashSync(user.password, 10);
    user.password = hash;

    const token = Token.getJwt(user.username);

    Users.add(user)
      .then(saved => {
        res.status(201).json({ id: saved.id, username: saved.username, token: token });
      })
      .catch(error => {
        res.status(500).json(error);
      })

  } else {

    res.status(400).json({
      message: 'Invalid user info, see errors',
      errors: validateResult.errors
    });
  }
})

module.exports = router;
```

- [ ] Don't forget to incorporate your validation middleware from before into each `post`, both login and register routers need this validation.

- [ ] Now go into your server and `.use` these new routes

```javascript
const express = require('express');
const cors = require('cors');
const helmet = require('helmet');

const logger = require('../middleware/logger');

const usersRouter = require("../users/user-router");
const loginRouter = require("../auth/login-router.js");
const registerRouter = require("../auth/register-router.js");

const server = express();

server.use(helmet());
server.use(logger);
server.use(express.json());
server.use(cors());

server.use("/api/login", loginRouter);
server.use("/api/register", registerRouter);
server.use("/api/users", usersRouter);

server.get('/', (req, res) => {
  res.send('<h1>🎣</h1>');
})

module.exports = server;
```

- [ ] Commit and push these changes

## Updates documentation

- [ ] On a new branch, make sure that each of your `/api/...` endpoints are dcumented well, including any body data they require, filtering they may offer, validation included, and what data each endpoint returns. Document this clearly and conscisely, so that it is very easy to browse. 

- [ ] Commit and push these changes

## Deploys

We're ready to merge these new changes into master now, so let's first go to Heroku and get our staging application ready.

- [ ] First, navigate to the `Resources` tab.
- [ ] Under `Add-ons` search for postgres and provision a new DB.

Once the Postgres DB is added, you'll be able to navigate to `Settings` in your Heroku staging application, and click on `Reveal Config Vars`. Here you'll notice that Heroku has added a `DATABASE_URL` for us. Cool! We'll need to add a few more Config Vars too while we're here.

- [ ] Add a `DB_ENV` key with a value of `production`
- [ ] Add a `JWT_SECRET` key with a secure value of your choosing.

- [ ] Now, merge your most recent changes into `master`, and check your Heroku Activity Feed for progress/errors
- [ ] Once the application is deployed, we'll need to run our migrations. There are a few ways to do this, but for simplicity's sake here we'll use Heroku's Consle that they provide under the `More` dropdown in the top right of your application dashboard. Choose `Run Console`
- [ ] run `knex migrate:latest`
- [ ] Using Postman or Insomnia, register a new user
- [ ] Using Postman or Insomnia, try to access `/api/users` with the token that is returned from `register`
- [ ] Once this staging application is running well, go into your `production` application and:
	- [ ] Provision a Postgres DB
	- [ ] Add `DB_ENV` and `JWT_SECRET` Config Vars
	- [ ] Manually deploy from `master`
	- [ ] Run your migrations
	- [ ] Test out production