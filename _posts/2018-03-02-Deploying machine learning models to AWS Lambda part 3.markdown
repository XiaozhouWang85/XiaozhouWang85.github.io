---
layout: post
comments: true
title:  "Deploying machine learning models to AWS Lambda (part 3)"
date:   2018-03-02 14:26:40 +0800
comments: true
categories: datascience machinelearning AWSLambda Deployment
---

The model that was developed in the last article is tricky to implement on AWS Lambda and I will discuss why later in this tutorial. To start with, I'll run through how to set up an AWS account and how to use AWS Lambda. As before, where others have already written good articles, I will link rather than re-create the content.

#### Why use cloud / AWS?
Amazon, Google, Microsoft and others all offer "cloud" services. In a nutshell, this means renting computing power in some format. More and more, applications are also introduced as part of the package. For large companies, this means they can outsource activities such as managing servers to the cloud provider who can typically do it far more efficiently. For individuals and startups, it allows them to rent computing power that they would have difficulty purchasing. For both, it allows 
them to increase or decrease their computing capacity on demand and avoid large fixed costs.

Amazon Web Services (AWS) has been around slightly longer than Google and Microsoft can be more mature in places. However, this is changing by the day. My main reason for opting for AWS is that it is the platform I've personally taken the time to learn.


#### Creating an AWS account

A credit card and mobile phone is needed to create an AWS account. AWS provides quite a generous free tier to many of their services. Generally if just trying out their services or using it for a personal site, no cost will be incurred. However, it is easy to accidentally spend money if you do not know what you are doing. Therefore, be careful to check the billing dashboard regularly. For a fairly detailed guide on getting an account see [here](https://tonyredhead.com/amazon-s3/create-aws-account).


#### Lambda and API gateway
The sheer number of services provided by AWS can be bewildering at first. This is very much in line with the idea that AWS provides lego-like components for a developer to build their applications with. 

![image]({{site.url}}/assets/aws_products.png){:class="img-responsive" width="100%"}

For this tutorial API gateway and Lambda will be used. This is not the only way to create an API on AWS but this approach has a number of benefits:
- Flexibility - only pay when functions is run, if infrequently used then will be cheaper than keeping a server up
- Scalability - if there is a sudden spike in API use then additional computing resources will be provisioned automatically by AWS

For data scientists this has a number of use cases. Champion / challenger models can be built and run on demand. Large but infrequent tasks can be turned into Lambdas and executed in parrallel. For example, a monte-carlo simulation can be run by sending each chunk to the API separately.

#### Building a trivial API
Now the introductions are done, we can move on to building a trivial API. This is shown below. It expects to recieve an integer number and if one is provided will respond with that number plus 5. The function should be called lambda_handler and input data is a dictionary called event.


```python
#Function
def lambda_handler(event, context):
    try:
        num=int(event['num'])
        return num+5
    except:
        return 0

#Test cases
test_1={"num":"15"}
test_2={"num":"test"}
```

We will turn the above function into a Lambda by using the AWS console. This is accessed from the web browser and allows functions to be deployed via a graphic user interface (GUI). All functions are also available from a command line interface (CLI). The GUI is extremely handy when trying out a new function but the CLI is a better choice for automating the steps. The steps are described in detail [here](https://www.fullstackpython.com/blog/aws-lambda-python-3-6.html).

Once the function is built, run the test cases and check the output. If successful, a screen as follows should be seen.

![image]({{site.url}}/assets/aws_lambda.png){:class="img-responsive" width="100%"}

#### Trigger Lambda function with API gateway
Now that the lambda function has been built, it has to be connected to the external world. This is done with API Gateway which handles incoming requests and parses these before passing them onto the Lambda function. For step by step instructions see [here](http://sebastianpatten.com/api-tutorial-amazon-api-gateway-part-2/#creating-api-gateway).

One area not covered in detail is how to configure the API gateway so that the JSON passes to Lambda in the correct way. 

In the Method Request section, define the query strings. In this case, only one is needed which is num.

![image]({{site.url}}/assets/aws_method_request.png){:class="img-responsive" width="75%"}

Then configure the Integration Request section as below. For each variable, create a mapping into the body.

![image]({{site.url}}/assets/aws_integration_request.png){:class="img-responsive" width="75%"}

Deploy the API by moving it to a stage. A URL will be provided to access the API.

![image]({{site.url}}/assets/aws_stage_deploy.png){:class="img-responsive" width="50%"}

After this is done, the API can now be accessed from anywhere. For example using the requests library as shown below.

```python
import requests

#Simple web scraper with dependency on Pandas
resp=requests.get(r'https://XXXXXXXXXX.execute-api.us-east-1.amazonaws.com/PRD/addfive',params={"num":"5"})
resp.text
'10'
```

This concludes this part 3 of the tutorial. We have set up an AWS account and then created a basic API that can be accessed via the internet. So far however, things have been easy as they only require standard python libraries. In order to use libraries such as Pandas or sci-kit learn we'll need to build our dependencies and package into a zip file. This will be covered in the next part.