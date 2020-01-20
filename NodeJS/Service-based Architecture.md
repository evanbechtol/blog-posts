# Node Service-oriented Architecture
Whether you are a beginner or an expert to Node.js, at the beginning of each project it's imperative that
you create a sound architectural landscape. This will enable you to grow your project while maintaining readability,
testability, and maintainability *(just to name a few [non-functional requirements](https://www.guru99.com/non-functional-requirement-type-example.html#2))*.

After reading this article, you'll be able: 
  1) Create an intuitive and clean project structure
  2) Properly document your code with the intention to improve usability and readability
  3) Understand the difference between concepts; controllers, loaders, services
  4) Create clean unit tests for your business logic


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
aren't an expert with them, just understand their importance and how the structure is able to enable us to utilize
these concepts.

* [SOLID](https://scotch.io/bar-talk/s-o-l-i-d-the-first-five-principles-of-object-oriented-design)
* [Separation of Concerns](https://medium.com/machine-words/separation-of-concerns-1d735b703a60)
* [Abstraction]()
* [Encapsulation]()
* [Unit Testing]()

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

Our service implements all of our logic and cna leverage the data access layer to
interact with the database! Once our logic reaches a result, we return the data, or an 
error if one occurred, to the controller.
```javascript
// services/PostService.js

const MongooseService = require( "MongooseService" ); // Data Access Layer
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
  async createPost ( postToCreate ) {
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

## Controller Layer

## Loaders

## Application Configurations

## Example Repository
