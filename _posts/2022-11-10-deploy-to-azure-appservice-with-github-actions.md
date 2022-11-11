---
title: Deploy to Azure with GitHub Actions
tags: Azure, GitHub Actions
layout: post
---

This article is going to introduce how to deploy an Angular Web application to Azure App Service with GitHub Actions.


Prerequisites
- Azure subscription, Resource Group and App Service
- Public GitHub repo

## Deploy code to Azure App Service with Github Actons

I use one of my public repo 【[demo repo](https://github.com/LiMeii/angular-ngrx)】 to demo how to deploy Angular web application to Azure App Service with GitHub Actions.


The first step is create one yml file under ```.github/workflows```:
```yml
name: Build and deploy Angular app to an Azure Web App

on:
  push:
    branches:
      - master

env:
  AZURE_WEBAPP_NAME: my-app-name   # set this to your application's name
  AZURE_WEBAPP_PACKAGE_PATH: '.'      # set this to the path to your web app project, defaults to the repository root
  NODE_VERSION: '16.x'                # set this to the node version to use

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

Before checkin this yml file, need config several variable: ```app-name``` ```publish-profile```(lines 66 and 68 of the code), these two variables are for Azure.


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


By following the steps above, you can build a GitHub Actions workflow that automates process CI/CD whenever there is code pushed into the master branch.


In real projects, We usually need to deploy on multiple environments , eg: ```Dev ``` ```Staging``` and ```Production```. How can we accomplish this? Find more details in this article: 


