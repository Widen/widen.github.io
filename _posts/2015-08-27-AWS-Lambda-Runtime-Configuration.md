---
title:  Runtime Configuration for AWS' Lambda
author: feltnerm
categories: aws lambda hack
published: true
permalink: /blog/2015/08/27/AWS-Lambda-Runtime-Configuration
---

A while ago Amazon released its [Lambda](http://aws.amazon.com/lambda/) service. Lambdas are simple, stateless  functions that get executed as callbacks to arbitrary AWS events.

One thing I disliked about Lambda is that you cannot provide any sort of runtime configuration to it. Because of this you are forced to build configuration into your source code either via the zip you give Lambda or through some other sort of mechanism.

Rebuilding and re-uploading a zip every time a configuration point changed was cumbersome, and having configuration baked into my Lambda seemed wrong. I wanted something like Elasticbeanstalk or ECS where I can just pass in environment variables and write my code to adapt.

The beauty of the Elasticbeanstalk/ECS model is that you do not have to write
or ship new code to deal with a changing configuration value. When your
database goes down would you rather deploy code to just change a value to a new endpoint?
I wanted this model with Lambda, and it seemed to fit perfectly, but alas it was not so.

Development was slow, and dynamically configured Lambdas were looking less likely as time wore on.

Until the day I realized Lambdas can have an optional `description` [when created](http://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/Lambda.html#createFunction-property). Also, Lambdas can use the [node aws-sdk](https://github.com/aws/aws-sdk-js) to make requests to other AWS web services, including Lambda itself.

I thought, "what if we used the description as configuration?".

So I filled the description of the Lambda with valid JSON. Then I wrote some code in the Lambda so it'd grab its own description when it fired up.

Here's an example of a Lambda that is processing its own description as a sort of runtime configuration.

``` javascript
// This function is called 'whatstheenv'
console.log('Loading event');

var aws = require('aws-sdk');

exports.handler = function(event, context) {

    var lambda = new aws.Lambda({apiVersion: '2014-11-11'});
    lambda.getFunctionConfiguration({ FunctionName: "whatstheenv" }, function(err, data){
        if (err) {context.done(err, err.stack); }
        var runtimeConfiguration = {};
        try {
            runtimeConfiguration = JSON.parse(data.Description);
        } catch (except) {
            console.log("unable to parse description as JSON");
        }
        console.log(runtimeConfiguration);
        context.done(null, "We are done here.");
    });

};
```

Now, we can simply edit the description in the console, and voilà, our next lambda run will have different data!

Obviously, we can use other sources such as SimpleDB, DynamoDB, S3, or GitHub as the source of our configuration, but using the lambda's description gives us a quick and easy way to get dynamic configuration.

The downside is that it is fairly easy for someone to blow away your changes by unknowingly editing them via the AWS Console.

As always, weigh the risks and benefits.
