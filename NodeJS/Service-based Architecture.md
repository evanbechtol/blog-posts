# Node Service-oriented Architecture
Whether you are a beginner or an expert to Node.js, at the beginning of each project it's imperative that
you create a sound architectural landscape. This will enable you to grow your project while maintaining readability,
testability, and maintainability *(just to name a few [non-functional requirements](https://www.guru99.com/non-functional-requirement-type-example.html#2))*.

After reading this article, you'll be able: 
  1) Create an intuitive and clean project structure
  2) Understand the difference between concepts; controllers, loaders, services
  3) Create clean unit tests for your business logic


## Table of contents
  0) Concepts
  1) Project Folder Structure
  2) 3-layer (Service-oriented) Architecture
  3) Service Layer
  4) Unit Testing
  5) Controller Layer
  6) Loaders
  7) Application Configurations
  8) Example Repository

## Concepts
Here are a few concepts that you should be acquainted with as you go through this article. Don't worry if you
aren't an expert with them, just understand their importance and how the structure enables us to utilize these concepts.

* [SOLID](https://scotch.io/bar-talk/s-o-l-i-d-the-first-five-principles-of-object-oriented-design)
* [Separation of Concerns](https://medium.com/machine-words/separation-of-concerns-1d735b703a60)
* [Abstraction](https://thevaluable.dev/abstraction-software-development/)
* [Encapsulation](https://stackify.com/oop-concept-for-beginners-what-is-encapsulation/)
* [Unit Testing](https://stackoverflow.com/a/1393/4515720)

## Project Folder Structure
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

## 3-layer Architecture
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

## Service Layer
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

## Unit Testing
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
## Controller Layer

## Loaders

## Application Configurations

## Example Repository
