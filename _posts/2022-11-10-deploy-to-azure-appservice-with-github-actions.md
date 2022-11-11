---
title: Deploy to Azure with GitHub Actions
tags: Azure, GitHub Actions
layout: post
---

This article is going to introduce how to deploy an Angular Web application to Azure App Service with GitHub Actions, and how to manual trigger GitHub workflow with workflow_dispatch.


Prerequisites
- Azure subscription, Resource Group and App Service
- Public GitHub repo

## Deploy code to Azure App Service with Github Actons

I use one of my public repo 【[demo repo](https://github.com/LiMeii/angular-ngrx)】 to demo how to deploy Angular web application to Azure App Service with GitHub Actions.


The first step is create one YML file under ```.github/workflows```:





<details>
    <summary><em>Click here to see YML file 👇</em></summary>


{::options parse_block_html="true" /}


```yaml
name: Build and deploy Angular app to an Azure Web App

on:
  push:
    branches:
      - master

env:
  AZURE_WEBAPP_NAME: my-app-name # set this to your application's name
  AZURE_WEBAPP_PACKAGE_PATH: '.' # set this to the path to your web app project, defaults to the repository root
  NODE_VERSION: '16.x'    # set this to the node version to use

jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Use Node.js ${{ env.NODE_VERSION }}
      uses: actions/setup-node@v3
      with:
        node-version: ${{ env.NODE_VERSION }}
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
      url: ${{ steps.deploy-to-webapp.outputs.webapp-url }}

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
          app-name: ${{ secrets.AZURE_WEBAPP_SERVICE_NAME }}
          slot-name: "production"
          publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
          package: ${{ env.AZURE_WEBAPP_PACKAGE_PATH }}
```

{::options parse_block_html="false" /}


</details> 

<br/>


Before checkin this yml file, need config several variable: ```app-name``` ```publish-profile```, these two variables are for Azure.


Open【[Azure Portal](https://portal.azure.com/)】, go to the ```Resouce Group``` and find the ```App Service```


```app-name``` means Azure App Service name: 
![azure-app-service-name](/assets/images/posts/github-actions/azure-app-service-name.png)


Download ```publish-profile``` from Azure Portal:
![azure-app-service-publish-profile](/assets/images/posts/github-actions/azure-app-service-publish-profile.png)


Get these two variables and put them into GitHub Secrets:
![github-actions-secrets](/assets/images/posts/github-actions/github-actions-secrets.png)


In the above yml file, I set trigger the workflow whenever code is pushed into master branch. For more workflow syntax can refer: 【[Workflow syntax for GitHub Actions](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)】


Now, checkin this yml file into master branch, and you can find the workflow under GitHub Actions:
![github-actions-workflow](/assets/images/posts/github-actions/github-actions-workflow.png)


Click into the workflow, you can see the two jobs: ```Build``` ```Deploy to Dev```
![github-actions-workflow-detail1](/assets/images/posts/github-actions/github-actions-workflow-detail1.png)


Click into the Job, you can see the detail steps:
![github-actions-workflow-detail-step](/assets/images/posts/github-actions/github-actions-workflow-detail-step.png)


Go to Azure Portal, you can browser the web application from here:
![auzre-portal-browse-web-app](/assets/images/posts/github-actions/auzre-portal-browse-web-app.png)


You also can check the deployed bundle file from here:
![auzre-portal-check-deployed-file](/assets/images/posts/github-actions/auzre-portal-check-deployed-file.png)


Click into ```App Service Editor (Preview)```:
![auzre-portal-check-deployed-file2](/assets/images/posts/github-actions/auzre-portal-check-deployed-file2.png)



## Manual trigger GitHub workflow with workflow_dispatch

By following the steps above, you can build a GitHub Actions workflow that automates process CI/CD whenever there is code pushed into the master branch. We also can manual trigger the workflow with ```workflow_dispatch```


<details>
    <summary><em>Click here to see YML file 👇</em></summary>

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
    - name: Use Node.js ${{ env.NODE_VERSION }}
      uses: actions/setup-node@v3
      with:
        node-version: ${{ env.NODE_VERSION }}
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
      url: ${{ steps.deploy-to-webapp.outputs.webapp-url }}

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
          app-name: ${{ secrets.AZURE_WEBAPP_SERVICE_NAME }}
          slot-name: "production"
          publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
          package: ${{ env.AZURE_WEBAPP_PACKAGE_PATH }}

```


</details> 


Checkin this change into master, and you can see the Manual trigger button:
![github-actions-worflow-manual-trigger](/assets/images/posts/github-actions/github-actions-worflow-manual-trigger.png)


Click ```Run workflow``` button, a workflow will run:
![github-actions-worflow-manual-trigger2](/assets/images/posts/github-actions/github-actions-worflow-manual-trigger2.png)



In real projects, We usually need to deploy on multiple environments , eg: ```Dev ``` ```Staging``` and ```Production```. How can we accomplish this? Find more details in my next article: 


