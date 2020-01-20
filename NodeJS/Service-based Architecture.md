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

## Service Layer

## Unit Testing

## Controller Layer

## Loaders

## Application Configurations

## Example Repository
