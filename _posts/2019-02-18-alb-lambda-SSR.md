---
layout: post
title: "Migration to ALB Lambda for server side rendering"
author: "Tylor Shin"
categories: frontend
tags: [SSR, Lambda, ALB, React, ALB Lambda]
image: Screenshotfrom2019-02-1713-51-28-480e55b2-7549-4cbf-870a-6ceb84591238.png
---

# Migration to ALB Lambda for server side rendering

At Nov, 2018. AWS announced [ALB can trigger Lambda function](https://aws.amazon.com/blogs/networking-and-content-delivery/lambda-functions-as-targets-for-application-load-balancers/). When I saw this news, I thought of that ALB can be attached to Route53. If it is possible, we can have HTTP Endpoint that triggers Lambda function without API Gateway service.

**TL:DR;**

1. If your service uses Lambda to serve SSR and also consists of CloudFront with API Gateway, I recommend migrating to ALB + Lambda.
2. However, If your service heavily relies on CDN cache for HTML(SSR) serving, preserving CF + API G might be better.
3. Migration doesn't change API that much, and helps debug.

---

## Background

Before the migration, I had felt big dissatisfaction about the architecture of the FrontEnd service. It consists of complicated AWS service layers.

![]({{ site.url }}/assets/img/old-architecture-1cb36915-6629-4b89-bc8b-e0dac152c512.png)

Old architecture of Scinapse

It needed CloudFront not for CDN service, but for the bridge between API Gateway and Route53. Because API Gateway can't be attached to Route53 directly.

This layer causes several side effects.

---

## Pros & cons of CloudFront

[Good]

- It provides CDN service features. However, Scinapse never needs CDN cache for HTML page. (because our cache rate is extremely low.)

[Bad]

- CloudFront is an unnecessary layer when we can connect Route53 to Lambda directly.
- It's difficult to debug user's request because CloudFront mangle HTTP Headers.
- It causes more cost. (CF, API G)

Especially I was in trouble to debug Server Side Rendering(SSR), I decided to change the architecture like below. (If it works.)

 

![]({{ site.url }}/assets/img/Screenshotfrom2019-02-1713-51-28-480e55b2-7549-4cbf-870a-6ceb84591238.png)

New architecture of Scinapse

I omit VPC or other settings not directly related with the current architecture.

As you can see, it's simpler and seems easy to set. 

---

## How to migrate

First of all, I recommend that you need to make exactly the same service with the different stage for the test. I have made `migrate` stage that copies production architecture.

If you're using Serverless Framework, you can do it simply by add deploy flag.

    $ serverless deploy --stage migrate  

(From now we'll use `migrate` stage only.)

**Step 1. Remove CloudFront && API Gateway**

After that, You need to remove CloudFront, API Gateway from your CloudFormation. If you're using Serverless, go to `serverless.yml` and remove all unneeded `Resources` relevant with CF & API G.

(You should remove `events - http` inside of `serverless.yml` function part.)  

**Step 1-2. Change Lambda Handler**

The handler function's request is little different with API Gateway trigger's request. You can find ALB's request object from [here](https://docs.aws.amazon.com/lambda/latest/dg/services-alb.html). 

Then also you should change the response. It's **BREAKING CHANGE**, so you have to set this up.

What you should do is insert `"isBase64Encoded": boolean` attribute. If it doesn't exist, Lambda throw error unlike with API Gateways's response. (In API Gateway response, it's optional.)

**Step 2. Make ALB**

Since there is no direct setting in Serverless until now, you should make it at AWS Console.

Go to EC2, and select Load Balancers  from the left column. Then create your Application Load Balancer with proper VPC & Security Group settings.

![]({{ site.url }}/assets/img/make-alb-ab1a0ba2-8325-4fa6-96d8-5123d0850d08.png)

Where to create Load Balancer

When you make the Load balancer, you should do some settings. (or you can add it later too)

1. Add HTTPS listener.

![]({{ site.url }}/assets/img/https-listener-ad72861f-569e-48f1-816b-39f1e92f63c8.png)

2. Configure Routing to Lambda Function.

3. Configure Registered Targets to the target Lambda function.

**Step 3. Connect Lambda with ALB**

![]({{ site.url }}/assets/img/add-trigger-3c98c2f1-fec7-44f4-93d8-29c88d817554.png)

Go to Lambda console, then add `Application Load Balancer` trigger.

**Step 4. Set up Route 53**

Connect Route53 for temporarily. Make A record for temporary domain(example: migrate.xxx.com) with Alias setting with your ALB.

Then try to connect your Route 53 setting!

If it works well, try to migrate your production server following above step too.

---

## Result

You can make CloudWatch Dashboard as before.

![]({{ site.url }}/assets/img/lambda-monitoring-62156cab-19b1-44eb-b17d-0137c7e7cd6d.png)

Monitoring Lambda relavant metrics

Also, You can monitor ALB too.

![]({{ site.url }}/assets/img/alb-monitoring-0a60e89a-94c2-43a8-90be-faaa0aaffd44.png)

ALB Monitoring metrics

---

## TIPS

You can add HTTP → HTTPS redirect logic (or other redirect logic) at ALB listener.

Go to ALB console, select target ALB, and add listener.

You can add 80 port listener for 301 redirect.

![]({{ site.url }}/assets/img/http-redirect-63d5c51e-1036-469a-a3da-bec3646262c6.png)

HTTP → HTTPS Redirect listener

That's it!

If you have any questions or ideas, please feel free to write comment for us!

Thanks.
