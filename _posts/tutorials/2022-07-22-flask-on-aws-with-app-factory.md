---
title: Deploy Flask on AWS Elastic Beanstalk with the App Factory
date: 2022-07-22 16:33:00
categories: HowTo
tags: aws, deploy, flask, guinicorn     # TAG names should always be lowercase
toc: true
---
## Intro

Hello! Here I'll show you how to deploy a flask app that uses the app factory pattern on the AWS Elastic Beanstalk service. I assume that:

- You followed [the official Flask Tutorial](https://flask.palletsprojects.com/en/2.1.x/tutorial/), which recommends the use of an app factory.
- You've already [created an account in AWS](https://portal.aws.amazon.com/gp/aws/developer/registration/index.html?nc2=h_ct&src=header_signup).
- You were able to deploy the sample app on the official [AWS Flask](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create-deploy-python-flask.html) Deployment tutorial*

> \* <sub>AWS EB Flask deployment tutorial uses EB CLI utility deploy, which is yet another tutorial to the queue. Although you can do it from the aws web console, I found it quite useful, as you can easily SSH on the server, inspect logs and deploy from the terminal.</sub>

## Problem

AWS Elastic Beanstalk (EB) is configured to look for a WSGI app called `application` inside it an `application.py` file. However, for those who wish to follow the official Flask documentation recommendation of using an App Factory, this default behaviour doesn't work, because there is no `application.py` file when using this pattern.

## Solution

In a the development environment, `flask run` runs the local server by implicitly calling the app factory `create_app()` in the main app's module (which is inside an `__init__.py` file). In production, you should use some WSGI Server like Gunicorn, which happens to be the default on the EB Python Platform. Knowing this is the key to solving this problem, because now the question is: how to explicitly call the `create_app()` with gunicorn? This question is easier to answer. On the flask documentation about [gunicorn deployement](https://flask.palletsprojects.com/en/2.1.x/deploying/gunicorn/) we have:

```bash
gunicorn -w 4 'hello:create_app()'
```

In the [AWS docs](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/python-configuration-procfile.html), we can find how it actually calls the gunicorn server:

```bash
web: gunicorn --bind :8000 --workers 3 --threads 2 'app-name:create_app()'
```

Then, we can use one of the tools that AWS EB provides for customizing the deploy: creating a Procfile. All you need to do is to add a `Procfile` to the root of the project to override the default:
```bash
web: gunicorn --bind :8000 --workers 3 --threads 2 'project.wsgi:application'
```

And that's it! Now you are good to go.

## Conclusion

It may seem a trivial problem for some experienced dev, but for those who are just beginning on the world of servers and deployment, I think this small problem can raise some important questions, given the fact that the key to solving it was knowing about the role of gunicorn (and reading some documentation, of course):

- What is WSGI in the first place? If both Werkzeug and Gunicorn have to do with it, what's the difference?

These will lead down to topics like CGI, FastCGI, HTTP/Web servers, Apache and apache mods, which I think are quite important to understand at least in an overview mode.
