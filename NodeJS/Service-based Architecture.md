# Node Service-oriented Architecture
Whether you are a beginner or an expert to Node.js, at the beginning of each project it's imperative that
you create a sound architectural landscape. This will enable you to grow your project while ensuring readability,
testability, and maintainability *(just to name a few [non-functional requirements](https://www.guru99.com/non-functional-requirement-type-example.html#2))*.

After reading this article, you'll be able: 
  1) Create an intuitive and clean project structure
  2) Understand the difference between concepts; controllers, loaders, services
  3) Create clean unit tests for your business logic


## Table of contents :bookmark_tabs:
  0) Concepts
  1) Project Folder Structure
  2) 3-layer (Service-oriented) Architecture
  3) Service Layer
  4) Unit Testing
  5) Controller Layer
  6) Loaders
  7) Application Configurations
  8) Example Repository

## Concepts :nut_and_bolt:
Here are a few concepts that you should be acquainted with as you go through this article. Don't worry if you
aren't an expert with them, just understand their importance and how the structure enables us to utilize these concepts.

* [SOLID](https://scotch.io/bar-talk/s-o-l-i-d-the-first-five-principles-of-object-oriented-design)
* [Separation of Concerns](https://medium.com/machine-words/separation-of-concerns-1d735b703a60)
* [Abstraction](https://thevaluable.dev/abstraction-software-development/)
* [Encapsulation](https://stackify.com/oop-concept-for-beginners-what-is-encapsulation/)
* [Unit Testing](https://stackoverflow.com/a/1393/4515720)

## Project Folder Structure :open_file_folder:
The below structure is what I use as a template in nearly all of my Node.js projects. This enables us to 
implement [Separation of Concerns](https://medium.com/machine-words/separation-of-concerns-1d735b703a60)
for our application.
```
src
│   index.js        # Entry point for application
└───config          # Application environment variables and secrets
└───controllers     # Express controllers for routes, respond to client requests, call services
└───loaders         # Handles all startup processes
└───middlewares     # Operations that check or maniuplate request prior to controller utilizing
└───models          # Database models
└───routes          # Express routes that define API structure
└───services        # Encapsulates all business logic
└───test            # Tests go here
```

## 3-layer Architecture :sparkling_heart:
Building on the principle of [Separation of Concerns](https://medium.com/machine-words/separation-of-concerns-1d735b703a60)
that we talked about earlier, the goal is to completely extract and separate our business logic from our API. Specifically,
we **never** want our business logic to be present in our routes or controllers. In the picture below, you'll see exactly 
how our application will flow. 
1) Controllers receive incoming client requests, and they leverage services 
2) Services contain all business logic, and can also make calls to the data access layer
3) The data access layer interacts with the database by performing queries
4) Results are passed back up to the service layer.
5) The service layer can then hand everything back to the controller
6) The controller can then respond to the client!

![3-layer Architecture](https://softwareontheroad.com/static/122dab3154cb7e417bbb210bbce7ca01/62eec/server_layers.jpg)

#### Question: Why can't I just place my business logic inside of my controller?
This is a great question! Because our routes are (in this case) created using the Express framework, there's 
a ton of extra fluff that is added to the `req` and `res` objects. If we want to test our business logic, we
are now tasked with the burden of creating a mock of those entire objects :cold_sweat:! By encapsulating all of our business logic
inside of services, we are able to test it without having to mock-up the Express `req` or `res` objects :relaxed:!

## Service Layer :factory:
The service layer encapsulates and abstracts all of our business logic from the rest of the application.

* **The Service Layer SHOULD**:
  * Contain business logic
  * Leverage the data access layer to interact with the database
  * Be framework agnostic
* **The Service Layer SHOULD NOT**:
  * Be provided the `req` or `res` objects
  * Handle responding to clients
  * Provide anything related to HTTP Transport layer; status codes, headers, etc.
  * Directly interact with the database
  
#### Example 
Here, our controller is importing the service that we will use to create posts.
Notice that we don't have any business logic in this file!

```javascript
// controllers/Post/index.js

const PostService = require( "../services/PostService" );
const PostServiceInstance = new PostService();

module.exports = { createCord };

/**
 * @description Create a cord with the provided body
 * @param req {object} Express req object 
 * @param res {object} Express res object
 * @returns {Promise<*>}
 */
async function createCord ( req, res ) {
  try {
    // We only pass the body object, never the req object
    const createdCord = await PostServiceInstance.create( req.body );
    return res.send( createdCord );
  } catch ( err ) {
    res.status( 500 ).send( err );
  }
}
```

Our service implements all of our logic and can leverage the data access layer to
interact with the database! Once our logic reaches a result, we return the data (or an 
error if one occurred) to the controller.
```javascript
// services/PostService.js

const MongooseService = require( "./MongooseService" ); // Data Access Layer
const PostModel = require( "../models/post" ); // Database Model

class PostService {
  /**
   * @description Create an instance of PostService
   */
  constructor () {
    // Create instance of Data Access layer using our desired model
    this.MongooseServiceInstance = new MongooseService( PostModel );
  }

  /**
   * @description Attempt to create a post with the provided object
   * @param postToCreate {object} Object containing all required fields to
   * create post
   * @returns {Promise<{success: boolean, error: *}|{success: boolean, body: *}>}
   */
  async create ( postToCreate ) {
    try {
      const result = await this.MongooseServiceInstance.create( postToCreate );
      return { success: true, body: result };
    } catch ( err ) {
      return { success: false, error: err };
    }
  }
}

module.exports = PostService;
```

## Unit Testing :heavy_check_mark:
Creating thorough tests for your code is essential for ensuring that your code is maintainable, and reliable.
If you follow the mindset of [Test-Driven Development (TDD)](https://www.agilealliance.org/glossary/tdd/), then you should
be creating unit tests **before** you begin writing any code. This enables us to ensure that we write the minimal amount
of code required to meet the requirements at hand, and once development is completed, our tests are already there!
![tdd flow](https://hackernoon.com/hn-images/1*PctatMK2pwPm2NrXuFs-ew.png)

There are many modules and ways to test your code, but in this example we will be using a combination of [mocha](https://mochajs.org/), [chai](https://www.chaijs.com/), and [nyc](https://www.npmjs.com/package/nyc).
This will give us a lot of flexibility for creating unit tests, and also let us know how much code is covered by our tests!

##### 1) To get started, add these 3 modules to our project
```npm i -s nyc mocha chai```

##### 2) Now create a new directory under the `test` directory for our `Post` tests
```
src
│   index.js        # Entry point for application
... // Other directories
└───test            # Tests go here
  └─── Post         # All tests for 'Posts' go here
       |  index.js
```

##### 3) Open `test/Post/index.js` and paste the following code
```javascript
const assert = require( "chai" ).assert;
const mocha = require( "mocha" );
const PostService = require( "../../services/PostService" ); // Import the service we want to test

mocha.describe( "Post Service", () => {
  const PostServiceInstance = new PostService();
  
  mocha.describe( "Create instance of service", () => {
     it( "Is not null", () => {
       assert.isNotNull( PostServiceInstance );
     } );
   
     it( "Exposes the createPost method", () => {
       assert.isFunction( PostServiceInstance.create );
     } );
   } );
} );
```

##### 4) Run the tests using the following command
```mocha test/* --reporter spec```

You should get output similar to this:
```
Post Service
       Create instance of service
         √ Is not null
         √ Exposes the createPost method
     2 passing (29ms)
```

Now, I know that we didn't do this in a true TDD fashion; we wrote the code *before* writing the test.
But because this article is more focused on the 3-layer architecture, I felt that it was important to first introduce
the concept of services before unit tests. Just know that if you were to put this into practice on your own,
it will benefit you greatly to write the tests prior to the code :sunglasses:.

Now that you have written your first tests, it's time to examine how we utilize our services in the Controller!

## Controller Layer :control_knobs:
The controller layer is responsible for handling client requests, and responding to them. Just to reiterate a very important point,
**this layer should never contain business logic!** We only leverage the services by passing the data that they need, not
the `req` or `res` objects themselves. This enables our services to remain framework agnostic!

I showed an example of the controller layer above, which you can also find here *(no need to re-invent the wheel)*.

```javascript
/**
 * @description Create a cord with the provided body
 * @param req {object} Express req object 
 * @param res {object} Express res object
 * @returns {Promise<*>}
 */
async function createCord ( req, res ) {
  try {
    // We only pass the body object, never the req object
    const createdCord = await PostServiceInstance.create( req.body );
    return res.send( createdCord );
  } catch ( err ) {
    res.status( 500 ).send( err );
  }
}
```

## Loaders :floppy_disk:
Loaders abstract all of our application startup processes into specific modules. This enables us to encapsulate and
maintain Separation of Concerns. If you dump everything into your application entry point, it gets cluttered very quickly.

To really drive this point home, compare this example with the one below it. Ask yourself which one would be easier
to maintain, which would be easier to scale and expand upon, which one would be easier to remove later on if no longer needed.

##### What not to do (BAD) :x:
```javascript
const bodyParser  = require( 'body-parser' );
const config      = require( './config' );
const express     = require( 'express' );
const morgan      = require( 'morgan' );
const path        = require( 'path' );
const routes      = require( './routes' );
const rfs         = require( 'rotating-file-stream' );
const compression = require( 'compression' );

let fs     = require( 'fs' ),
    logDir = path.join( __dirname, config.logDir );

// Check for 'logs' directory
fs.access( logDir, ( err ) => {
  if ( err ) {
    fs.mkdirSync( logDir );
  }
} );

// Initialize express instance, and log rotation
let app             = express(),
    accessLogStream = rfs( 'access.log', {
      interval : '1d',
      path     : logDir
    } );

// Setup views and pathing
app.set( 'view engine', 'html' );
app.set( 'views', path.join( __dirname, 'public' ) );

// Serve static content
app.use( express.static( path.join( __dirname, 'public' ) ) );
app.use( express.static( path.join( __dirname, 'node_modules' ) ) );

// Set up middleware
app.use( morgan( 'dev', { stream : accessLogStream } ) );
app.use( compression() );
app.use( bodyParser.urlencoded( {
  extended : false,
  limit    : '20mb'
} ) );
app.use( bodyParser.json( { limit : '20mb' } ) );

// Pass app to routes
routes( app );

// Start application
app.listen( config.port, () => {
  console.log( 'Now listening on', config.port );

} );
```

##### What to do (GOOD) :100:
```javascript
const config = require( "./config" );
const mongoose = require( "mongoose" );
const logger = require( "./services/Logger" );

const mongooseOptions = {
  useCreateIndex: true,
  useNewUrlParser: true,
  autoReconnect: true
};

mongoose.Promise = global.Promise;

// Connect to the DB an initialize the app if successful
mongoose.connect( config.dbUrl, mongooseOptions )
  .then( () => {
    logger.info( "Database connection successful" );

    // Create express instance to setup API
    const ExpressLoader = require( "./loaders/Express" );
    new ExpressLoader();
  } )
  .catch( err => {
    //eslint-disable-next-line
    console.error( err );
    logger.error( err );
  } );
```

It's very easy to tell which file is more maintainable, readable, scalable, etc. Now, to show you what the loader looks like.
In the example above, you'll notice we have one loader, the `ExpressLoader`. Here is how the loader is structured

```javascript
const bodyParser = require( "body-parser" );
const express = require( "express" );
const morgan = require( "morgan" );
const path = require( "path" );
const routes = require( "../routes" );
const compression = require( "compression" );
const logger = require( "../services/Logger" );
const config = require( "../config" );

class ExpressLoader {
  constructor () {
    const app = express();

    // Setup error handling, this must be after all other middleware
    app.use( ExpressLoader.errorHandler );

    // Serve static content
    app.use( express.static( path.join( __dirname, "uploads" ) ) );

    // Set up middleware
    app.use( morgan( "dev" ) );
    app.use( compression() );
    app.use( bodyParser.urlencoded( {
      extended: false,
      limit: "20mb"
    } ) );
    app.use( bodyParser.json( { limit: "20mb" } ) );

    // Pass app to routes
    routes( app );


    // Start application
    this.server = app.listen( config.port, () => {
      logger.info( `Express running, now listening on port ${config.port}` );
    } );
  }

  get Server () {
    return this.server;
  }

  /**
   * @description Default error handler to be used with express
   * @param error Error object
   * @param req {object} Express req object
   * @param res {object} Express res object
   * @param next {function} Express next object
   * @returns {*}
   */
  static errorHandler ( error, req, res, next ) {
    let parsedError;

    // Attempt to gracefully parse error object
    try {
      if ( error && typeof error === "object" ) {
        parsedError = JSON.stringify( error );
      } else {
        parsedError = error;
      }
    } catch ( e ) {
      logger.error( e );
    }

    // Log the original error
    logger.error( parsedError );

    // If response is already sent, don't attempt to respond to client
    if ( res.headersSent ) {
      return next( error );
    }

    res.status( 400 ).json( {
      success: false,
      error
    } );
  }
}

module.exports = ExpressLoader;
```

What we have done is abstracted our startup logic for express into a single file. This enables us to easily remove/replace
the framework later on if we choose to. This also makes it much more easy to debug or track down issues, rather than having to 
check in multiple places, or a monster file.

## Application Configurations :clipboard:
You should take great care to never expose your application secrets and configurations. The last thing that you want
is for a malicious entity to have all the back doors flung wide-open for them to do whatever they want! There are
some fantastic modules out there; [dotenv](https://www.npmjs.com/package/dotenv) which can give your some amazing functionality to protect your app secrets.
But for the sake of simplicity, we are going to create an `index.js` file in our `config` directory. This will hold
all of our application configuration parameters for us, which we can use in our other files.

```javascript
// config/index.js

const config = {
  dbUrl: process.env.DBURL || "mongodb://localhost/test-db",
  port: process.env.PORT || 3000,
  env: process.env.NODE_ENV || "development",
  logDir: process.env.LOGDIR || "logs",
  viewEngine: process.env.VIEW_ENGINE || "html"
};

module.exports = config;
```  

It's that simple! then you can simply import the file wherever you need it, and reference the variables.

## Example Repository :gift:
You can find an example repo that demonstrates how this structure can be implemented on [my GitHub](https://github.com/evanbechtol/bare-express)

## References and Resources :gem:
I can't take credit for everything here, and definitely got some help! Please check out these other resources that inspired me.

* [Bulletproof Node.js Project Architecture](https://dev.to/santypk4/bulletproof-node-js-project-architecture-4epf#configs)
* [Scotch.io](https://scotch.io/bar-talk/s-o-l-i-d-the-first-five-principles-of-object-oriented-design)
* [Stackify](https://stackify.com/oop-concept-for-beginners-what-is-encapsulation/)
* [Agilealliance](https://www.agilealliance.org/glossary/tdd/)
* [Hackernoon](https://hackernoon.com/hn-images/1*PctatMK2pwPm2NrXuFs-ew.png)
