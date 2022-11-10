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
![github-actions-workflow1](/assets/images/posts/github-actions/github-actions-workflow1.png)

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


