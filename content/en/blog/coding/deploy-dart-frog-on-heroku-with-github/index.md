---
author: 
    - Gabriel
cover: "cover.jpg"
date: 2022-09-17T02:04:06-05:00
series: ["Dart Frog"]
tags: ["Dart Frog", "Devops", "GitHub Actions", "Heroku", "Dart", "Pipeline"]
title: "Deploying with ease Dart Frog services"
subtitle: "Deploy your Dart Frog web service on Heroku via GitHub actions"
---

### Intro

When it comes to **developer experience**, Pipelines are your best friend. They allow you to automate your workflow and make it easier to deploy your code.
In this article, we will see how to deploy our Dart Frog web service on **Heroku** with **GitHub actions**.

### Why Heroku
Heroku is a cloud platform as a service (PaaS) that **enables us developers to build, run, and handle web applications entirely in the cloud**.
It is a great way to deploy your web service without having to worry about the infrastructure, which comes handy when either you don't have the time or the expertise to play with DEVOPS stuff.
The real reason why we are using Heroku, tho', is that **it can be used for free**, which is awesome for our hobby projects or MVPs.

### Why GitHub actions
There is no engineer who isn't somehow familiar with GitHub. It's one of the most popular git services you can find, and it is a great way to collaborate with other developers.
GitHub actions is a feature that **allows us to automate our workflow**. It is a great way to build, test and deploy our code.

### Deployment
#### Init
Before starting make sure you have a **GitHub account** and a **Heroku account**.
I'll also give for granted that you already have an empty **GitHub repository** and a **Heroku app** created.

As first step create a **sample Dart Frog** project.
- `$ dart_frog create <heroku_deployment_project> && cd <heroku_deployment_project>`

Let's push it to our GitHub repository.
- `$ git init`
- `$ git add .`
- `$ git commit -m "Initial commit"`
- `$ git remote add origin git@github.com:<User>/<RepositoryName>.git`
- `$ git push -u origin master`

#### Dockerfile

By default, when you build a Dart Frog project, the CLI creates a Dockerfile file in the `/build` folder (After running `$ dart_frog build`) so you can easily build a container and deploy your app. 

We are not going to use it tho', it doesn't quite work with Heroku.

So, let's create a **new Dockerfile** in the root folder of our project.

- `$ touch Dockerfile`

Now, let's add the following content to it:

```Dockerfile
# Official Dart image: https://hub.docker.com/_/dart
# Specify the Dart SDK base image version using dart:<version> (ex: dart:2.17)
FROM dart:latest AS build

WORKDIR /app

# Copy entire repo from the build machine to the image.
COPY . .

# Resolve app dependencies.
RUN dart pub get
RUN dart pub global activate dart_frog_cli

# Build the app.
RUN export PATH="$PATH":"$HOME/.pub-cache/bin" && dart_frog build

# Compile the app to a native executable.
RUN dart compile exe ./build/bin/server.dart -o ./build/bin/server

# Start server.
CMD ["./build/bin/server"]
```

#### The pipeline
Now that we have our Dockerfile in place, we can start creating the pipeline.

First thing we'll need to add some secrets to our GitHub repository:
- **HEROKU_API_KEY**: You can find it in your Heroku account, under `Account > Settings > API Key`
- **HEROKU_EMAIL**: Your Heroku email (Used to register the account)
- **HEROKU_APP_NAME**: The unique name of your Heroku app

To add the secrets, go to your repository settings, and click on `Secrets > New repository secret`.


Create a new directory .github/workflows folder file called main.yml and put it under :
- `$ mkdir -pv .github/workflows && touch .github/workflows/main.yml`

And add the following content to the file:

```yaml
name: Deploy

on:
  push:
    branches:
      - master

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: akhileshns/heroku-deploy@v3.12.12
        with:
          heroku_api_key: ${{secrets.HEROKU_API_KEY}}
          heroku_email: ${{secrets.HEROKU_EMAIL}}
          heroku_app_name: ${{secrets.HEROKU_APP_NAME}}
          usedocker: true
          dontuseforce: 1
```

Let's push our code to GitHub.
- `$ git add .`
- `$ git commit -m "Add Dockerfile and pipeline"`
- `$ git push`

**NOTE:** Before pushing make sure the automatic deployment is **disabled** in your Heroku app settings.

![automatic deployment](automatic_deployment.png "Automatic Deployment on Heroku")


#### Deploy
Once we perform the push, GitHub will start the pipeline and deploy automatically our app to Heroku, which will take a few minutes.
You can check the progress at `https://github.com/<User>/<RepositoryName>/actions`.
Once your build is done go to your Heroku account and enjoy your successfully deployed app.

Everytime you push in your branch `master`, from now on, GitHub will automatically deploy your app to Heroku. If you want to enable a different trigger have a look at the [documentation](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows).

### Conclusion
In this article we saw how to deploy our Dart Frog web service on Heroku with GitHub actions: the process overall is quite simple, far away from being rocket science.
It can be easily adapted to other projects and you can use it to deploy any app, but you'll have to modify a little the Dockerfile.

Feel free to fork, contribute or reach out: https://github.com/Hyla96/heroku_dart_frog

DEMO: https://dart-frog-deployment-example.herokuapp.com/