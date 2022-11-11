---
title: Deploy on multiple environment with GitHub Actions
tags: Azure, GitHub Actions
layout: post
---

This article is going to introduce: 
- Deploy on multiple environment with GitHub Actons
- Add reviewers approve a workflow deploying
- Reuse workflow


Prerequisites
- Azure subscription, Resource Group and App Service
- Public GitHub repo


## Deploy on multiple environment with GitHub Actons

In the previous article【[Deploy to Azure with GitHub Actions](https://limeii.github.io/2022/11/deploy-to-azure-appservice-with-github-actions/)】have introduced how to deploy web application to Azure App Service with GitHub Actions. Now what? In this article I am going to introduce how to deploy on multiple environment with GitHub Actions.


In our real projects, we usually need deploy on multiple environment, eg: ```Development``` ```Staging``` ```Production```, actually this is quite simple, just add two more jobs in the YML file, once the build job is complete, deploy the artifacts to different environment(App Service).


{::options parse_block_html="true" /}


<strong>Click here to see YML file 👇</strong>

<details>

```yaml

name: Build and deploy Angular app to an Azure Web App

on:
  push:
    branches:
      - master

  workflow_dispatch:
    inputs:
      logLevel:
        description: 'Log level'
        required: true
        default: 'warning'
        type: choice
        options:
        - info
        - warning
        - debug
      tags:
        description: 'Test scenario tags'
        required: false
        type: boolean
      environment:
        description: 'Environment to run tests against'
        type: environment
        required: true

env:
  AZURE_WEBAPP_NAME: my-app-name  # set this to your application's name
  AZURE_WEBAPP_PACKAGE_PATH: '.' # set this to the path to your web app project, defaults to the repository root
  NODE_VERSION: '16.x'           # set this to the node version to use

jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Use Node.js ${ { env.NODE_VERSION } }
      uses: actions/setup-node@v3
      with:
        node-version: ${ { env.NODE_VERSION } }
        cache: "npm"
        cache-dependency-path: package-lock.json
        
    - name: npm install, build, and test
      run: |
        npm install
        npm run build --if-present
    
    - name: Zip artifact for deployment
      run: |
        cd dist
        zip release.zip ./* -r

    - name: Upload artifact for deployment job
      uses: actions/upload-artifact@v3
      with:
        name: node-app
        path: ./dist/release.zip

  deployDev:
    name: Deploy to Dev
    permissions:
      contents: none
    runs-on: ubuntu-latest
    needs: build
    environment:
      name: "Development"
      url: ${ { steps.deploy-to-webapp.outputs.webapp-url } }

    steps:
      - name: Download artifact from build job
        uses: actions/download-artifact@v3
        with:
          name: node-app

      - name: unzip artifact for deployment
        run: unzip release.zip

      - name: "Deploy to Azure WebApp"
        id: deploy-to-webapp
        uses: azure/webapps-deploy@v2
        with:
          app-name: ${ { secrets.AZURE_WEBAPP_SERVICE_NAME } }
          slot-name: "production"
          publish-profile: ${ { secrets.AZURE_WEBAPP_PUBLISH_PROFILE } }
          package: ${ { env.AZURE_WEBAPP_PACKAGE_PATH } }

  
  deployProd:
    name: Deploy to Production
    permissions:
      contents: none
    runs-on: ubuntu-latest
    needs: deployDev
    environment:
      name: "Production"
      url: ${ { steps.deploy-to-webapp-prod.outputs.webapp-url } }

    steps:
      - name: Download artifact from build job
        uses: actions/download-artifact@v3
        with:
          name: node-app

      - name: unzip artifact for deployment
        run: unzip release.zip

      - name: "Deploy to Azure WebApp"
        id: deploy-to-webapp-prod
        uses: azure/webapps-deploy@v2
        with:
          app-name: ${ { secrets.AZURE_WEBAPP_SERVICE_NAME_PROD } }
          slot-name: "production"
          publish-profile: ${ { secrets.AZURE_WEBAPP_PUBLISH_PROFILE_PROD } }
          package: ${ { env.AZURE_WEBAPP_PACKAGE_PATH } }
```


</details> 

<br/>


{::options parse_block_html="false" /}


Checkin this change into master, and you can see the workflow deploy to ```Development``` ```Production```:
![github-actions-workflow-deploy-on-multi-env](/assets/images/posts/github-actions/github-actions-workflow-deploy-on-multi-env.png)


We also can see the manual trigger, can choose different environment:
![github-actions-workflow-deploy-on-multi-env2](/assets/images/posts/github-actions/github-actions-workflow-deploy-on-multi-env2.png)


## Add reviewers approve a workflow deploying

By following the steps above, you may notice that every time we checkin code into master, workflow triggered, and this will automatically deploy to both ```Devlopment``` and ```Production```. 


Generally, we don't deploy every checkin to  ```Production```, we may need to complete a few features and test them in ```Development``` or ```Staging```, if there are no issues, then we deploy to ```Production```. 


There two ways can achieve this:
- Create two different worflow: one is for ```Development```, in this workflow YML file, listen to push code to **master** branch, every checkin to master, trigger this workflow deploy to ```Development```; the other one is for ```Production```, in this workflow YML file, listern to push code to **release** branch, every checkin to release, trigger this workflow deploy to ```Production```

- Use the same YML file with above, in the workflow listen to push code to **master** barnch, but we need to add reviewers for Production job, that means every checkin to master, it will automatically tirgger workflow build and deploy to ```Development```, but need reviewer to approve production deploying.

In this article, I use the second solution, add reviewers approve a workflow deploying. Go to the GitHub repo settings, add one new environment: Production

![github-actions-workflow-deploy-production1](/assets/images/posts/github-actions/github-actions-workflow-deploy-production1.png)

<br/>

![github-actions-workflow-deploy-production2](/assets/images/posts/github-actions/github-actions-workflow-deploy-production2.png)


<br/>

Add required reviewers and save protection rules:
![github-actions-workflow-deploy-production3](/assets/images/posts/github-actions/github-actions-workflow-deploy-production3.png)


Manual triggr a workflow run and you will see ```Deploy to Production``` job pending on the approval from reviewers:
![github-actions-workflow-deploy-production7](/assets/images/posts/github-actions/github-actions-workflow-deploy-production7.png)


And the reviewers also will receive below notification email:
![github-actions-workflow-deploy-production5](/assets/images/posts/github-actions/github-actions-workflow-deploy-production5.png)


Reviewers click ```Approve and deploy``` will trigger the Production deployment
![github-actions-workflow-deploy-production6](/assets/images/posts/github-actions/github-actions-workflow-deploy-production6.png)


## Reuse workflow

By fowllowing the steps above, you may notice, the deployment job is kind of duplicated, in this demo code, we only deploy to two environment. In real Projects, there may have 4 environemts, if follow above steps, all these 4 deployment job will be duplicated. From the best practice prespective, we need extract the duplicated deployment job into one invidual YML file, and reuse this deploy YML workflow.


We can use ```strategy``` to create a reuseable matrix job for deployment.


The first step is extract the deployment job to one invidual YML file:


{::options parse_block_html="true" /}


<strong>Click here to see YML file 👇</strong>

<details>

```yaml
name: Reusable deployment workflow

on:
  workflow_call:
    inputs:
      target-env:
        required: true
        type: string
    secrets:
      AZURE_WEBAPP_SERVICE_NAME:
        required: true
      AZURE_WEBAPP_PUBLISH_PROFILE:
        required: true
      AZURE_WEBAPP_SERVICE_NAME_PROD:
        required: true
      AZURE_WEBAPP_PUBLISH_PROFILE_PROD:
        required: true



jobs:
  deploy:
    name: Deploy to ${ { inputs.target-env } }
    permissions:
      contents: none
    runs-on: ubuntu-latest
    environment:
      name: ${ { inputs.target-env } }
      # url: ${{ steps.step_id.outputs.url_output }}

    steps:
      - run: echo "🎉 target evn ${ { inputs.target-env} }"
      - run: echo "🎉 target evn inputs.target-env"
      - run: echo "💡 get azure webapp name from secrets ${ { secrets.AZURE_WEBAPP_SERVICE_NAME } }"
      - run: echo "🍏 is Dev  ${ { inputs.target-env } } == 'Development'"
      - run: echo "🐧 is Prod  ${ { inputs.target-env } } == 'Production'"

      - name: Download artifact from build job
        uses: actions/download-artifact@v3
        with:
          name: node-app

      - name: unzip artifact for deployment
        run: unzip release.zip

      - name: "Deploy to Azure Dev WebApp"
        if:  inputs.target-env == 'Development'
        id: deploy-to-webapp-dev
        uses: azure/webapps-deploy@v2
        with:
          app-name: ${ { secrets.AZURE_WEBAPP_SERVICE_NAME } }
          slot-name: "production"
          publish-profile: ${ { secrets.AZURE_WEBAPP_PUBLISH_PROFILE } }
          package: '.'

      - name: "Deploy to Azure Prod WebApp"
        if:  inputs.target-env == 'Production'
        id: deploy-to-webapp-prod
        uses: azure/webapps-deploy@v2
        with:
          app-name: ${ { secrets.AZURE_WEBAPP_SERVICE_NAME_PROD } }
          slot-name: "production"
          publish-profile: ${ { secrets.AZURE_WEBAPP_PUBLISH_PROFILE_PROD } }
          package: '.'


```


</details> 

<br/>


{::options parse_block_html="false" /}



The second step is update previous YML file as:


{::options parse_block_html="true" /}


<strong>Click here to see YML file 👇</strong>

<details>

```yaml

name: Build and deploy Angular app to an Azure Web App

on:
  push:
    branches:
      - master
  workflow_dispatch:
    inputs:
      logLevel:
        description: 'Log level'
        required: true
        default: 'warning'
        type: choice
        options:
        - info
        - warning
        - debug
      tags:
        description: 'Test scenario tags'
        required: false
        type: boolean
      environment:
        description: 'Environment to run tests against'
        type: environment
        required: true

env:
  AZURE_WEBAPP_NAME: my-app-name # set this to your application's name
  AZURE_WEBAPP_PACKAGE_PATH: '.' # set this to the path to your web app project, defaults to the repository root
  NODE_VERSION: '16.x'          # set this to the node version to use

jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Use Node.js ${ { env.NODE_VERSION } }
      uses: actions/setup-node@v3
      with:
        node-version: ${ { env.NODE_VERSION } }
        cache: "npm"
        cache-dependency-path: package-lock.json
        
    - name: npm install, build, and test
      run: |
        npm install
        npm run build --if-present
    
    - name: Zip artifact for deployment
      run: |
        cd dist
        zip release.zip ./* -r

    - name: Upload artifact for deployment job
      uses: actions/upload-artifact@v3
      with:
        name: node-app
        path: ./dist/release.zip

  ReuseableMatrixJobForDeployment:
    needs: build
    strategy:
      fail-fast: true
      matrix:
        target: [Development, Production ]
    uses: ./.github/workflows/deployment.yml
    with:
      target-env: ${ { matrix.target } }
    secrets: inherit

```


</details> 

<br/>


{::options parse_block_html="false" /}


Checkin above changes into master, and you can see the workflow run:

![githhub-workflow-reuse](/assets/images/posts/github-actions/githhub-workflow-reuse.png)

<br/>

![githhub-workflow-reuse2](/assets/images/posts/github-actions/githhub-workflow-reuse2.png)


In development deploying job, the ```Deploy to Production``` job skipped:
![githhub-workflow-reuse6](/assets/images/posts/github-actions/githhub-workflow-reuse6.png)



 ```Deploy to Production``` job pending on reviewers' approval:

 ![githhub-workflow-reuse3](/assets/images/posts/github-actions/githhub-workflow-reuse3.png)


Reviewer approve production deploying:

![githhub-workflow-reuse4](/assets/images/posts/github-actions/githhub-workflow-reuse4.png)

In production deploying job, the ```Deploy to Development``` job skipped:
![githhub-workflow-reuse5](/assets/images/posts/github-actions/githhub-workflow-reuse5.png)