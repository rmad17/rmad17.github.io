+++
title = 'Application Deployment Checklist'
date = 2024-04-20T10:18:04+05:30
draft = true
+++


### Your app is built and you need to deploy it. What Next? Read On...

Before we deploy applications we need to follow a set of process to ensure we are prepared with enough information to manage any issue which may or may not be critical.
Over the years I have experienced these scenarios and I have created a set of my personal checklists which allow me to quickly check 
that my application meets all the requirements for recovering from potential pitfalls. 

Broadly these are the following points we will be covering:
1. Error Tracking.
2. Secrets Management.
3. Environment Variables.
4. Dockerfile(s).
5. Log Setup.
6. Static Storage.
7. Database Setup.
8. Application Infrastruture.


#### __Error Tracking__
> `To err is human`

We will continue to make mistakes and errors will continue forever. Whats the next best thing?
To capture the error quickly. 
There are a bunch of error tracking tools available in the market. My preferred choice has been to use Sentry over the years and it has never let me down. You are free to choose the tool of your choice but adding an error tracker should be top of the list.

#### Secrets Management
