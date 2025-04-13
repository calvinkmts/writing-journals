# Improving DAGs Development for MWAA

## Background

Our team relies on AWS Managed Workflow for Apache Airflow (MWAA) to automate our data transformation jobs. However, testing our DAGs locally remains a challenge since we lack a standard process for it. We face the classic issue: without local testing, we are exposed to push buggy code to production, debugging is slow, and surely slow down development as we don't have the instant feedback to our code, especialy as our code is tightly coupled with external call to S3 services and lacking the unit tests. Unless this issue is highlighted, there's a little incentive to improve the workflow. 

Initially, I encountered DAG development through a Lambda job that used Jinja templates for DAG creation. However testing this Lambda code locally was also a challenge, which led to a convoluted workflow. Our process for verifying changes was something like this:

1. Code the lambda that creates the DAG.
2. Push the code to git repository so it gets picked up by CI/CD and synced to AWS Lambda.
3. Run the Lambda code to check if anything breaks.
4. If everything 'appears' fine, check the MWAA environment for any issues from the new DAG.

It's a long process a long process yet it doesn't promote correctness that can be quickly and easily validated.

## The Problems

I want to improve my workflow for MWAA, especially since this is my first time working with this tool. I'm trying to avoid a scenario where my changes will causes issues in the remote environment, especially in production. Ideally, I should be able to develop a DAG or even generate multiple DAGs using a DAG factory, and receive immediate, green-light feedback, and only then push the code to the git repository. Streamlining this process will not only boost development speed but also enhance code quality and operational stability.

## The Solutions

### The Search for Locally Deploying Airflow

I believe when working with Airflow, the ideal approach is to deploy it locally so that I can test my DAGs thoroughly before pushing any changes. Here are few methods I evaluated:

> **A Note on Prioritization:**
> Time was of the essence. I needed a solution that could be set up and deployed locally with minimal overhead. I didn't have the luxury to delve deeply into every option; instead, my priority was to quickly establish a workflow that provides instant clarity on where and how the code is applied, without having to navigate complex configurations.

#### 1. The pip Install Solution

The [Airflow Documentation](https://airflow.apache.org/docs/apache-airflow/2.3.0/start/local.html) explains how to install Airflow locally using pip. However, this method is constrained by the active Python version on your system. Although using virtual environments (venv) can help, the extra setup isn't always seen as a viable option, especially when business pressure demand quick results.

#### 2. The dockerfile solution

[The Dockerfile Solution](https://airflow.apache.org/docs/apache-airflow/2.3.0/start/docker.html) is more appealing since it encapsulates a specific version of Airflow and isolates it from your system. Meaning if something breaks, it's easier to identify and address the issue. My colleague, was one of the first to experiment with Docker in our team has been tinkering with this approach. However, I found that it still doesn't fully emulate MWAA locally in a ready-to-use manner.

#### 3. Finding aws-mwaa-local-runner

Initially, I was exploring more effective ways to handle AWS Lambda, I was looking into tools like the AWS Serverless Application Model (SAM) that promises easier local development of Lambda functions. During my search, I discovered that AWS provides additional tools to simplify local development. This led me to [aws-mwaa-local-runner](https://github.com/aws/aws-mwaa-local-runner), which at first glance demonstrate how simple the local deployment is.

- It includes a dedicated `dags` folder where you can place your DAGs or DAG factory code.
- An accompanying ENV file called `.env.localrunner` that lets you specify your AWS credentials so the tool can interact with your AWS credentials (use with caution to avoid unexpected cloud bill).
- It comes with its own CLI that manages starting and building the local Airflow environment.

Ultimately, aws-mwaa-local-runner became my solution for testing both individual DAGs and my DAG factory code locally.

### Next Step: The CLI app

We don't want to develop the DAGs or the DAG factory code one time and then manually copying it to the dags folder and see if it fails then back to the code and then copying it again. So my solution to the problem is to setup a little cli app. The context here is, I was working working with a python monorepo project, in some sense there will be some cli automation at some points to manage codes across projects, which I decide let's add testing the DAG factory code to the cli app. 

So I created the requirements:

1. I want to be able to `git clone` the aws-mwaa-local-runner repository.
2. I want to be able to switch versions for the aws-mwaa-local-runner, in this case it's managed through tagging in the their GitHub Repository.
3. I want to be able to easily copy my DAGs or my DAG factory code easily, this is important because say if the team picked up the project and it turns out testing locally is such a chore then why do it, that's the idea of this little cli app and why I want push this test locally paradigm.
4. I want to be able to call the existing cli from my cli that already exists in the aws-mwaa-local-runner.

Once these requirements were defined, the rest was a matter of coding. 

Below is an example illustrating the Python monorepo project structure:

```
python_monorepo/
├── cli/              # Contains custom CLI tools and automation scripts
├── common/           # Share modules, utilities, and configurations
├── dagfactory/       # Houses code for generating and managing DAGs
├── project_a/        # Contains specific business logic for Project A
├── project_b/        # Contains specific business logic for Project B
└── project_c/        # Contains specific business logic for Project C
```

### Improvement

Now that I may not always working with the this codebase, but I believe this is like the idea if in case I needed to use aws-mwaa-local-runner again.

Since I might not always be working with this particular codebase, I'd like to design a new approach to ensure that the solution remains reusable in case I need to use aws-mwaa-local-runner again. Here's the new requirements:

1. Everything from the old requirements. 
2. I want this cli to be packaged for easy download and execution on UNIX-based system (something like .sh script). Additionally, by adding the CLI tool to the system PATH, it can be used as a background/system utility, making it easier to be used from any location, independent from any specific project.

I believe this is a good improvements to provide a robust and portable solution for quickly testing whether my code changes will work before they are pushed to remote environment.

By installing the CLI tool in the system PATH, developers can quickly invoke it from anywhere on the system. This approach promotes reusability across multiple projects and ensures testing becomes a more simple process.

## Conclusions 

While local testing of DAGs, especially when using a DAG factory, I recognize that integrating unit testing could further enhance testability. Unit tests offer a granular approach to verify that individual components meet their specification or not, Which is essentials for catching issues early on. However, implementing unit tests is a topic in itself.

For now, the current solution allows me to run integration tests effectively and locally. These tests help ensure that any changes I make do not break the existing functionality, therefore mantaining a stable and reliable remote environment.

