+++
title = 'Application Deployment Checklist'
date = 2024-04-20T10:18:04+05:30
draft = true
+++


> ```Your app is built and you need to deploy it. What Next? Read On...```

Before we deploy applications we need to follow a set of process to ensure our application stack is prepared for any expected or unexpected events.
Over the years I have experienced these scenarios and I have created a set of my personal checklists which allow me to quickly check 
that my application has all the requirements for recovering from potential pitfalls. This article is about sharing the same.

```
Broadly these are the following points we will be covering:
1. Error Tracking
2. Secrets and Environment Variables
3. Dockerfile(s)
4. Log Setup
5. Application Infrastruture
6. Observability Platforms
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

#### Log Setup
Logs are the foot soldiers of any technical investigation. They reveal the most boring yet vital information about any system or application.
Hence, logs are critical to any system you deploy. The question is "how do I log"? Well, my usual answer to this is log to an external system that has good search capabilities. Logging to a .txt file maybe good for a dev system for production systems its not of much help unless we are able to quickly search what we need. Common recommened tools include [Logstash](https://www.elastic.co/logstash), [GrayLog](https://graylog.org/).

#### Application Infrastrure
We usually setup infrastructure following the easiest approach and this may serve us well in the short term, however, may force us to spend a lot more time in the long term. When allocating infrastructure for production systems we need to ensure that the infrastructure must be able to handle sudden surges in traffic. For this it is essetial to atleast setup an AutoScalingGroup(ASG) if using AWS or equivalent in our cloud providers. We should also look to setup Subnets and VPCs first, Databases second and the rest of the stack after that. This sequence is based on the dependency of the resources on each other.

#### Observability Platform
Once the application infrastructure is setup, it also needs to be monitored. Effective monitoring with the help of alarms ensures we are able to quickly respond to any issues arising with the resources. Popular observability platforms include, [AWS Cloudwatch](https://aws.amazon.com/cloudwatch/), [Prometheus](https://prometheus.io/), [Datadog](https://www.datadoghq.com/), etc.

#### CI/CD Platforms
The best time to setup a CI/CD process is when setting up deployment. This will often save hundreds of developer hours in future as well as improve the quality of code that is deployed by adding test runs for unit tests, linting checks, etc. Common examples of CI/CD tools are [Jenkins](https://www.jenkins.io/), [CircleCI](https://circleci.com/), [AWS CodePipeline](https://aws.amazon.com/codepipeline/), etc.


### Conclusion
Now, we've come to the end of our checklist and your app is ready for deployment to production. I hope this checklist helps you in ensuring the deployment process is smooth and its future ready to avoid any performance issues for the near future.
