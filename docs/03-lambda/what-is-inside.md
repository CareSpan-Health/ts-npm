---
sidebar_position: 1
---

# What is inside?

API packages are used for handling everything related to building a serverside endpoint

## Middlewares

In order to build a lambda that has the same output, we need to have some middlewares for reinforcement

## Database Models

Building the APIs will need to connect to CareSpan database for user verification (Do I have access to patient A?)

APIs often need to pull data out from CareSpan main/ehr database and classes are built to do that easily

APIs may also need to connect to other database for tracking

## Input (Query string / Post body) helpers

APIs will always have inputs to consider. Helpers can be used to manage the inputs from GET/POST/others

## AWS S3 helpers

There are times when we need to save data somewhere. One example is the S3 and we have S3 helper classes to do that
