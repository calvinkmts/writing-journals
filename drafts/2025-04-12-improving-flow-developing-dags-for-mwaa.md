# Improving Workflow: Developing DAGs for MWAA (Managed Workflow for Apache Airflow)

## Background

Our team is using Airflow to automate transformation jobs, in this case we are using the managed service by AWS called Managed Workflow for Apache Airflow or MWAA for short. Now the issue at hand is that we don't have the consensus on how to test our DAGs locally. It's traditional issues sometimes it may not be the case sometime it is, as I believe if we can test things locally then we will easily test bugs or some thing we don't want in the production. 

My first introduction with the DAGs code is through a lambda job. We have this lambda where it has the templates using jinja on how the DAGs will look like, with config it will just loop everything and then for each pipeline or DAG it will create each new DAG file.

First, another issues arise we don't actually have the way to test this lambda code locally. Which means our flow is... 

1. Code the lambda which is used to create the DAG.
2. Push the code to git repository so then it will be picked up by CI/CD and be synced to AWS Lambda.
3. Run the Lambda code to check if something will breaks.
4. If everything is fine, check the MWAA environment for any issues from the new DAG.

It's a long process...

## The Question

Now I want to improve this flow especially for MWAA as this is the first time I actually in contact with the tools. I don't really want this way of work to bite me in the ass later on. I want to be able to create a DAG or even in this case a DAG factory code and then check if it's all green, then all is well then push the code to the git repository. 

## The Solutions

### The Search for Locally Deploying Airflow

I think if we work with Airflow then the best way to work with it is to be able deploy it locally and put everything into it. 

#### 1. The pip Install Solution

If you take a look at the [Airflow Documentation](https://airflow.apache.org/docs/apache-airflow/2.3.0/start/local.html) then you can see that it is locked in to the python version currently active of course there is better way then directly using current python such as (venv), but I don't want to really spend time trying out things just to test locally, which most of the time it's not accepted with the business or even your superior.

#### 2. The dockerfile solution

[The Dockerfile Solution](https://airflow.apache.org/docs/apache-airflow/2.3.0/start/docker.html) is something more attracting because you can see the airflow version and it's not really attached to your system directly, so if something breaks then it's more easy to isolate where the issue is. My collegue is actually the one that is using this flow (he is the first one that requested access to Docker) and I have seen him tinkering with it. However I still not satisfied because As you can see that it's not really like ready-to-use to replicate MWAA on local.

#### 3. Finding aws-mwaa-local-runner

I was looking to handling AWS Lambda better, until I found that AWS has some tools to create lambda easily (supposedly) with AWS Serverless Application Model (SAM). I notice that AWS has some tools to like help us developer create easier on local, with a little bit of search pointing me to [aws-mwaa-local-runner](https://github.com/aws/aws-mwaa-local-runner) which in brief really pointing out how easy it is to use

- You can see there is dags folder for you to put your DAGs or DAG factory code
- Dig deeper then you can see an env file where you can list your aws credentials so it can interact with your AWS resources (beware: use with caution if you don't want to rake in some unexpected cloud bill)
- It has it's cli to start and build it.

Well, later this is my solution to test out DAGs and my DAG factory code. 

### Next Step: The CLI app

Okay, we don't really want to develop our code one time and then manually copying it to the DAG and see if it fails then back again and then copying it again. So my solution to the problem is to setup a little cli app. The context is we are working with a python monorepo project. so, in some sense there will be some cli automation at some points to managing codes across projects. why not do it with the testing tools. 

So I created the requirements:

1. I want to be able to git clone the aws-mwaa-local-runner.
2. I want to be able to switch versions for the aws-mwaa-local-runner, in this case it's managed through tagging in the their GitHub Repository.
3. I want to be able to easily copy my DAGs or my factory DAG code easily, this is important because say if the team picked and it turns out testing locally is such a chore then why do it, that's the idea of this little cli app.
4. I want to be able to call the existing cli from my cli that already exists in the aws-mwaa-local-runner.

Well after that everything is just code. 

### Improvement

Now that I may not always working with the this codebase, but I believe this is like the idea if in case I needed to use aws-mwaa-local-runner again.

Here's the new requirements:

1. Everything from the old requirements. 
2. I want this cli to be easily downloaded and then run on UNIX based system (something like .sh files).

I believe this is a good solution in case I need to quickly test whether my codes will work or not. 

## Conclusions 

Asides from the testing DAGs locally, especially for DAG factory I believe unit testing will be one way to improve that testability, but I believe it's another topic. With this I can already do a integation tests as the unit tests will help developer with testing if their changes breaks the current requirements or not. 

