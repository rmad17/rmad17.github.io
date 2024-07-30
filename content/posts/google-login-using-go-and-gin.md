+++
title = 'Google Login Using Go and Gin'
date = 2024-07-30T22:48:24+05:30
draft = false
cover.image = '../../go-google-cover.png'
cover.hidden = false
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
Run the command `go get -u github.com/gin-gonic/gin` to add gin as dependency.
Lets now create an app.go file. This is the main app of the gin project and runs the application. 
```go

package main

import "github.com/gin-gonic/gin"

func main() {
	router := gin.Default()
	router.GET("/hello", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "world",
		})
	})
	router.Run() // listen and serve on 0.0.0.0:8080
}
```
You can now run your server by running the command `go run app.go`.
Now to test our API we can run `curl http://localhost:8080/hello` and we should see the following response - 
```json
{"message":"world"}
```

---------------------------------------------------------------------------------------

### Add Google OAuth 

Now that our Gin application is up we will work on the authentication with Google OAuth. The API flow will work in the following manner:
1. Load the page with Google Sign In Button which triggers the beginning of the authentication process. For our usecase this will use the `/auth/google/login` route.
2. Initiate the authentication process using the gothic handler which invokes the Google APIs. For this process we will use the `/auth/google/begin` route.
3. The final step is the callback which is received from Google and is based on the configuration in Google Cloud Console. Here, we have defined the route as `/auth/google/callback`.

We also need to create a Google Client ID and Client Secret. 
You need to follow below steps once you open Google API Console
 * From the project drop-down, select an existing project, or create a new one by selecting Create a new project
 * In the sidebar under "APIs & Services", select Credentials
 * In the Credentials tab, select the Create credentials drop-down list, and choose OAuth client ID.
 * Under Application type, select Web application.
 * In Authorized redirect URI use `http://localhost:8080/auth/google/callback`
 * Press the Create button and copy the generated client ID and client secret


Next, create a directory call `templates` and add an `index.tmpl` with the following the code. 
```html
<!-- templates/index.html -->
<!doctype html>
<html>
<head>
    <title>Google SignIn</title>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/css/bootstrap.min.css">
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/font-awesome/4.7.0/css/font-awesome.min.css">
    <style>
        body        { padding-top:70px; }
    </style>
</head>
<body>
<div class="container">
    <div class="jumbotron text-center text-success">
        <h1><span class="fa fa-lock"></span> Social Authentication</h1>
        <p>Login or Signup </p>
        <a href="/auth/google/begin" class="btn btn-danger"><span class="fa fa-google"></span> SignIn with Google</a>
    </div>
</div>
</body>
</html> 
```

Next we will add a `success.tmpl` template file in the same templates directory.
```html
<!-- templates/success.html -->
<!doctype html>
<html>
  <head>
    <title>Google SignIn</title>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/css/bootstrap.min.css">
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/font-awesome/4.7.0/css/font-awesome.min.css">
      <style>
          body        { padding-top:70px; }
      </style>
  </head>
  <body>
    <div class="container">
      <div class="jumbotron">
          <h1 class="text-success  text-center"><span class="fa fa-user"></span> Profile</h1>
          <div class="row">
            <div class="col-sm-6">
                <div class="well">
                        <p>
                            <strong>Id</strong>: {{.UserID}}<br>
                            <strong>Email</strong>: {{.Email}}<br>
                            <strong>Name</strong>: {{.Name}}
                        </p>
                </div>
            </div>
        </div>
      </div>
    </div>
  </body>
</html>
```

Now that the templates are ready we need to add the routers and controllers for the same. We start with adding the dependencies:
```bash
go get github.com/markbates/goth
go get github.com/markbates/goth/gothic
go get github.com/markbates/goth/providers/google
go get github.com/google/gofuzz
go get github.com/gorilla/sessions
go get github.com/joho/godotenv
```

We also need to load our environments for our Google Client ID and secret loading from env variables. Lets create a `loadEnvs.go` in `core` module. We will load this in our app.go.
```go
package core

import (
	"log"

	"github.com/joho/godotenv"
)

func LoadEnvs() {
	err := godotenv.Load()
	if err != nil {
		log.Fatal("Error loading .env file")
	}

}
```

Next, we add our controllers code which performs the actual authentication. Add the following code to `googleauth.go`.
```go
package controllers

import (
	"net/http"

	"github.com/gin-gonic/gin"
	"github.com/gorilla/sessions"
	"github.com/markbates/goth/gothic"
)

func GoogleOAuthLogin(c *gin.Context) {
	c.HTML(http.StatusOK, "index.tmpl", gin.H{
		"title": "Google Login",
	})
}

func GoogleOAuthBegin(c *gin.Context) {
	key := "Secret-session-key" // Replace with your SESSION_SECRET or similar
	maxAge := 86400 * 30        // 30 days
	isProd := false             // Set to true when serving over https

	store := sessions.NewCookieStore([]byte(key))
	store.MaxAge(maxAge)
	store.Options.Path = "/"
	store.Options.HttpOnly = true // HttpOnly should always be enabled
	store.Options.Secure = isProd

	gothic.Store = store
	q := c.Request.URL.Query()
	q.Add("provider", "google")
	c.Request.URL.RawQuery = q.Encode()
	gothic.BeginAuthHandler(c.Writer, c.Request)
}

func GoogleOAuthCallback(c *gin.Context) {
	user, err := gothic.CompleteUserAuth(c.Writer, c.Request)
	if err != nil {
		c.AbortWithError(http.StatusInternalServerError, err)
		return
	}
	c.HTML(http.StatusOK, "success.tmpl", user)
}
```

Finally we add the routers in our `app.go` as shown below and restart our application server.
```go
package main

import (
	"myginapp/controllers"
	"myginapp/core"
	"os"

	"github.com/gin-gonic/gin"
	"github.com/markbates/goth"
	"github.com/markbates/goth/providers/google"
)

func main() {
	core.LoadEnvs()
	google_client_id := os.Getenv("GOOGLE_OAUTH_CLIENT_ID")
	google_client_secret := os.Getenv("GOOGLE_OAUTH_CLIENT_SECRET")
	google_provider := google.New(google_client_id, google_client_secret, "http://localhost:8080/auth/google/callback", "email", "profile")
	router := gin.Default()
	router.GET("/hello", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "world",
		})
	})
	goth.UseProviders(google_provider)
	router.LoadHTMLGlob("templates/*")
	router.GET("/auth/google/login", controllers.GoogleOAuthLogin)
	router.GET("/auth/:provider/begin", controllers.GoogleOAuthBegin)
	router.GET("/auth/:provider/callback", controllers.GoogleOAuthCallback)
	router.Run() // listen and serve on 0.0.0.0:8080
}
```

Once this is done, visit the login page at `/auth/google/login` and click on `Signin with Google` button. This should start the authentication process and when completed successfully it would return an access token with `user` object. This `AccessToken` can then be used for authenticating other APIs.

I hope this article is helpful in getting started with Google SignIn using Gin. If you have questions or further doubts please feel free to add in comments.
