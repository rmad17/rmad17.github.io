+++
title = 'Google Login Using Go and Gin'
date = 2024-07-26T07:48:24+05:30
draft = true
tags = ["go", "golang", "gin", "google login", "auth", "social auth", "beginner friendly"]
categories = ["tech", "auth", "golang", "gin", "google-login"]
disqus_url = 'https://souravbasu.xyz/posts/google-login-using-go-gin/'
disqus_identifier = '2024-07-26T00:16:26+05:30'
+++

In this blog we will learn about how to get started with Golang and create a auth with Google Login. 
We will be using the [Gin](https://gin-gonic.com) and gorm ORM. During my initial uses of Gin I felt for new users the documentation is not friendly enough, hence I decided to write this article.
If you are coming from a Python/Ruby world Gin is **NOT** like Django or Rails. Rather, its more like Flask and Sinatra. This is how the framework folks defines Gin:

### What is Gin?

> Gin is a web framework written in Golang.
> It features a Martini-like API, but with performance up to 40 times faster than Martini.
> If you need performance and productivity, you will love Gin.


*Gin supports the following features:*
 * API Performance - Has a very small memory footprint and radix tree based routing.
 * Middleware Support - Supports chain of middlewares to modify an incoming HTTP request.
 * Crash Safety - Can capture panic during request processing and recover from it.
 * JSON Validation - Inbuilt JSON parsing and validation.
 * Route Grouping - Provides us with an option of grouping multiple routes rogether.
 * Error Management - Provides us with an easy way to collect errors and use them in middlewares if required.
 * Rendering - Has an inbuilt renderer that can be used with JSON and HTML.
 * Extendable - Can extend functionality with middlewares.

---------------------------------------------------------------------------------------

### Creating our first Gin app

Now that we know a little bit about Gin lets get started with our app.

Assuming Go is installed in the system, create a directory `myginapp` and run the command `go mod init myginapp`. If Go is not already installed follow the instructions from [here](https://go.dev/doc/install).
The command creates the `go.mod` file and is required to run our application.
