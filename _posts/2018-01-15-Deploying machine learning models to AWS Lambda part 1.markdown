---
layout: post
title:  "Deploying machine learning models to AWS Lambda (part 1)"
date:   2018-01-15 13:21:40 +0800
comments: true
categories: datascience machinelearning AWSLambda Deployment
---

Python is great for machine learning but one frequently encountered challenge is how to deploy these models into production. This tutorial will explore how a model built with sci-kit learn can be deployed to run on AWS Lambda. While there are many other ways of deploying a machine learning model, AWS Lambda benefits from being "severless". In a nutshell, what this means is that instead of paying AWS for a server by the hour, charging is per execution of the function. This is great for infrequently running processes but is also an easy way to build distributed computing applications. Once a function is turned into an API, many can be called in parallel making it perfect for easily distributed applications like Monte Carlo simulations. 

First a disclaimer, I'm not a developer and have almost no experience running critical processes in production. In fact, the main motivation for writing this tutorial was the fact that many articles on deploying machine learning models and on AWS assume the reader is already an experienced developer. 

I recently went about learning to deploy a machine learning model to run on AWS Lambda and found the process fairly painful. This article is designed to help those building models but without much experience as developers get their models into production. This can be to demo a prototype model or for production use where the use cases are not business critical. If like me, you are not an experienced developer then I would suggest you apply caution in this area and try as much as possible to understand what you don't know. There is a reason why people get paid to do this full time!

Pre-requisites:

 * Reader should already know how to build machine learning models
 * Readers should be familiar with sci-kit learn, pandas and numpy 

Tutorial Sections:

 * Introduction
 * Building a model and running it on a local machine
 * Running a trivial model on AWS Lambda
 * Creating a zipped deployment package for Lambda
 * Slimming down deployment packages
