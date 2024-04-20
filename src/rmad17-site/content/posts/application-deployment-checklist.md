+++
title = 'Application Deployment Checklist'
date = 2024-04-20T10:18:04+05:30
draft = true
+++


> ```Your app is built and you need to deploy it. What Next? Read On...```

Before we deploy applications we need to follow a set of process to ensure we are prepared with enough information to manage any issue which may or may not be critical.
Over the years I have experienced these scenarios and I have created a set of my personal checklists which allow me to quickly check 
that my application meets all the requirements for recovering from potential pitfalls. 

```
Broadly these are the following points we will be covering:
1. Error Tracking
2. Secrets and Environment Variables
3. Dockerfile(s)
4. Log Setup
5. Static Storage
6. Application Infrastruture
7. CI/CD Pipelines
```
****

#### Error Tracking
Mistakes happen and errors will follow. Whats the next best thing? -- to capture the error quickly. 
There are a bunch of error tracking tools available in the market. My preferred choice has been to use Sentry over the years and it has never let me down. You are free to choose the tool of your choice but adding an error tracker should be top of the list.

#### Secrets and Environment Variables
One of the common amateur mistakes is leaving secrets out in the code. It could be your thirdparty key, it could be a salt for password hashing or something else. Leaving it in the code or creating a .env file in production system is not a good idea. Some one could go back the git commit history and find out a secret key that might have been committed in the past. 
What if the instance crashes and you lose your .env file? There are also chances of losing env variables for one environment. These kind of issues could be chaotic to say the least and could dent reputation at its worst. 
The solution is to use a 3rd party tool to store and mange the secrets. Examples of such tools are [Hashicorp Vault](https://www.vaultproject.io/), [AWS Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html), [AWS Secrets Manager](https://aws.amazon.com/secrets-manager/). These tools directly/indirectly support adding environment specific variables and allows these values to be fetched over an API. Among the three Vault comes with OSS option while Paramter Store offers the cheapest hosted solution.

#### Dockerfile(s)
Its 2024 and you really need to use Docker if you are already not doing so. For those who are unfamiliar with Docker, Dockerfiles allow us define the whole development environment in on single file. There are times when we may need more than one service for eg. a web and worker. For these scenarios docker-compose.yml is your friend. Docker compose allows us to combine multiple Dockerfiles and create multiple services which can be run from one single file - `docker-compose.yml`. 
To learn more about docker and docker-compose follow the links below. \
https://docs.docker.com/guides/get-started/ \
https://docs.docker.com/compose/ 
