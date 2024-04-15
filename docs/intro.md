---
sidebar_position: 1
---

# NPM Packages

## Intro

This is the documentation to dive into the following NPM packages

* General Package
  * For handling CareSpan objects and common functions with value objects
  * For handling Common functions
* Singleton - For managing state, events, communication where singleton is required across other packages/applications
* API Package - For building lambda endpoints

## How do they fit together

### General Package

The "General Package" can be used in both front end and backend to handle CareSpan object. CareSpan stores [data](https://carespan-health.github.io/dataset) in a very specific way and the data structure is published here. However, these data may not always be easy to parse through. Therefore, you may download this "General Package" to help parse the CareSpan data so you have an interface.

For example, `UserPatient` is a Value Object to handle a patient. There is a need to get the home phone number. You may use `UserPatient->getFirstName()` instead of trying to figure out whether you need to pull from `fname` field (which may not be obvious)

"General Package" also has common functions. For example a common function that is used a lot in CareSpan is how to change the date time from UTC to Local. So, there these functions are also made available in "General Package" to help with data manipulation.

In general, "General Package" can be used for both front-end and back-end as this is not client/sever side dependent.

### Singleton Package

We have singleton functions and adding to the packages can be tricky. To keep it very simple, we have a separate NPM package so it can be added as a `dependecies`.

### API Package

This is for building lambdas. To build lambdas, we need a common set of middleware (to ensure the lambdas are built consistently), connector to the databases, and access to other services (e.g. AWS S3, Pusher)

|                                           | Front End | Back End | Common |
| ----------------------------------------- | --------- | -------- | -------- |
| [General](./category/general-package)     | X         | X        |  |
| [Singleton](./category/singleton-package) | /         | X        | This mostly for backend |
| [API](./category/api-package-lambda)      |           | X        |  |
