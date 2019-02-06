# CICD with Travis CI [![Build Status](https://travis-ci.org/simvergara/cicd-travis.svg?branch=master)](https://travis-ci.org/simvergara/cicd-travis)

This project is to demonstrate a Continuous Integration/Continuous Deployment (CICD) Pipeline being used to lint, test, build, and deploy an angular app to Pivotal on the fly. 
It was initally created to demonstrate at McMaster University. We will be using the angular Tour of Heroes tutorial application as 
a sample and Travis CI as a CICD tool.


* [Code Setup](#code-setup)
* [GitHub Setup](#github-setup)
* [Travis CI](#travis-ci)
   * [Basic Configuration](#basic-configuration)
   * [Continuous Testing](#continuous-testing)
   * [Continuous Deployment](#continuous-deployment)
* [Pivotal Setup](#pivotal-setup)
* [Next Steps](#next-steps)
* [References](#references)

## Code Setup

To set up our codebase, create a new Angular application using the Angular CLI
```
ng new cicd-travis
cd cicd-travis
```
This initiates the starter application that you can run locally on [http://localhost:4200](http://localhost:4200) by using the command `ng serve`. 
You can run a linter on the application using `ng lint`. The application also has simple unit tests set up that can be run using `ng test`. 

## GitHub Setup

Create a GitHub repo with the name `cicd-travis`

## Travis CI

You can register to [Travis CI](https://travis-ci.org) by using your GitHub account. Once logged in, it will pull up a list of all of your repositories and you can select the ones that you would like to enable Travis CI.

Once you enable a repository, Travis CI will look for a `.travis.yml` file in the root of your project. This file will have all the configurations that Travis CI will use for building and deploying your application. For the final `.travis.yml` file take a look [here](https://github.com/simvergara/cicd-travis/blob/master/.travis.yml)

### Basic Configuration
For the purpose of this project, the basic `.travis.yml` file will look like this:
```
language: node_js
node_js:
  - "9"
dist: trusty
sudo: required

branches:
  only:
  - master

before_script:
  - npm install -g @angular/cli

script:
  - ng lint
  - ng build --prod --base-href https://cicd-travis.cfapps.io
```

The file follows standard yml format. This file is doing the following:
1. Use **Node.js**
2. Watch only the master branch. Any changes made to master will kick off a build.
3. Install the Angular CLI before doing anything else.
4. Run the linter with `ng lint`
5. Build the application. (Note the `--base-href` is referring to my instance of Pivotal Cloud Foundry)

### Continuous Testing

A big part of CICD is testing. Before building or deploying an application, it is best to test it. We need to tell Travis CI to test the application, so in order to do this we add a line to the `script` section of the `.travis.yml` config that looks like this:
```
- ng test --watch=false --browsers=ChromeHeadless
```
What this does is perform the unit tests that the angular application has set up. When running these tests, we are also specifying the `--browsers=ChromeHeadless` option. This is for when it isn't feasible to run a full chrome instance, and comes in useful when testing the application on a remote server such as with Travis CI.

### Continuous Deployment

At this point, we have a configuration that will lint, test, and build our project on any changes made to the master branch. That's useful, but ideally we would also like to integrate and deploy this code to the existing application. For this project, we deploy to our pivotal instance (more details explained in the **Pivotal Setup** section). 

We need to tell Travis CI that we will be deploying to pivotal and how to do it. In order to do this, add the following to the `.travis.yml` configuration file:
```
deploy:
   provider: cloudfoundry
   username: simon.vergara.1993@gmail.com
   password: $PIVOTAL
   api: https://api.run.pivotal.io
   organization: simvergara
   space: development
```

You will notice above that I use `$PIVOTAL` as the password parameter. This is an environment variable that you can set on Travis CI. On travis-ci.org, go to the Settings tab of your repository. Near the bottom you will be able to set an environment variable and a value. It will be injected at the runtime. 

Note: Travis CI also supports encryption of your variables. For more information, see [Encryption Keys](https://docs.travis-ci.com/user/encryption-keys/). This would be used when you want a sensitive value to be set in the `.travis.yml` file.

For full details on deploying to CloudFoundry, see [CloudFoundry Deployment](https://docs.travis-ci.com/user/deployment/cloudfoundry/).

## Pivotal Setup

Create an account at [https://login.run.pivotal.io](https://login.run.pivotal.io). You will need to verify your email, activate your trial, and set up your organization and space. 

In order to push your code to your pivotal instance, you will need to have a `manifest.yml` file. Like the `.travis.yml` file, the `manifest.yml` file contains the configuration, but in this case for pushing your application to the cloud. Our `manifest.yml` file will look like the following:
```
---
applications:
- name: cicd-travis
  buildpacks:
    - staticfile_buildpack
  path: dist/cicd-travis
```
By the time we are looking to deploy our application to pivotal, it has already been built. We use the `manifest.yml` to specify the following:
1. The name of our application in our pivotal instance
2. The buildpacks that we want to use (in this case, `staticfile_buildpack` is appropriate since our application has already been built).
3. The path where the application is sitting. When the application is built, the static files are placed in the `/dist/cicd-travis` folder.

The `manifest.yml` file usually follows a standard format specified by CloudFoundry. More information can be found at [Deploying with App Manifests](https://docs.cloudfoundry.org/devguide/deploy-apps/manifest.html)

## Next Steps

Your CICD pipeline is now built. Any future commits to the `master` branch will go through the process of linting, testing, building, and deploying automatically using Travis CI. When you create a pull request to your master branch, Travis CI will see the request and run the linting, testing, and building parts of the pipeline, however it will stop before it deploys since the code is not yet in the master branch. 

As you develop and make changes to your code, keep in mind to make any necessary changes to your `.travis.yml` and `manifest.yml` files.

## References

[The Angular DevOps Series: CT/CI with Travis CI and GitHub Pages](https://blog.angularindepth.com/the-angular-devops-series-ct-ci-with-travis-ci-and-github-pages-3c02664f078)
