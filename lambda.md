## A. Basic Lambda concepts

``AWS lambda`` is an **event-driven** compute service i.e in order to invoke it or run the code, **you need to pass an Event, a JSON formated document**.

* **Event** is a JSON formated document(not necessary a JSON object) that contains data for a lambda function to process. So an Event can be:
    * JSON Array
    * JSOn Object
    * Appying String
    * An Integer
    * Null

* **[Context](https://docs.aws.amazon.com/lambda/latest/dg/python-context.html)** : A **context object** is passed to your function by lambda services. It contains informations about the
    * Invocation, 
    * Function itself and 
    * Runtime environment.

* The **handler** is the entry mpoint of your lambda function. When the function is invoked, th e **runtime** passed  **two arguments** to the function
    * Event and  
    * Context

    > You can  have **multiple Python function** in your Lambda but only one **handler**

    > You can also cretae folder 

You can decide with function can be the **handler** by using the **handler properties**:
> **name_of_the_file.handler_function** and pass   **event** and **context** to the **handler_functtion**.
>
> The **handler** need to return an object, that can be **serialzable** into **JSON**
``````python
return {
    'statuscode': 200,
    'body' : json.dumps('Hello from lambda')
}
``````
* We can return an **Exeption**
``````python
return {
    'statuscode': 200,
    'body' : Exeption()
}
``````
* We can return an **String**
``````python
return {
    'statuscode': 200,
    'body' : 'Hello'
}
``````
* We can return an **null**
````python
return {
    'statuscode': 200,
    'body' : null
}
````
# B. Intermediate Lambda concepts
``AWS lambda`` environment variables **allow you to modify thr function behavior without updating the code** and we can access environment variable from code. All you need is to :

````python
1. import os
2. Access your variable with os.environ['VARIABLE_NAME']
````
>  In our ``AWS lambda`` function, we are able to use any standard Python librairies such as JSON, OS, etc ... without any problem. **That's because these librairies are included in the Python Librairies itself**.

> But if you want to use a third party librairies such as **requests**, you need **to create a deployment package**.

To create a deployment package without any dependency:
1. Open a ``terminal``.
2. Create a folder name ``My_function`` (for example).
3. ``cd`` to that folder.
4. Create a file(for example ``lambda_function.py``)
5. Open file and write your desire lambda function.
6. The deployment need to be in **the zip format**.
7. ````zip  deployment-package.zip lambda_function.py````
8. Upload the zip file from AWS console and ond't **forget to rename the lambda handler**.

To create a ddeployment package with dependencies:
1. Repeat previous 1. to 6.steps.
2. Install the requests packages in the same directry by creating a virtual environment.
3. ````zip  -r deployment-package.zip .````
4. Upload the zip file from AWS console and ond't **forget to rename the lambda handler**.

We have our function bundle with the request dependencies.  It's very likely that **the majority of Lambda we will create in the future, will also require the request librairy**. We could bundle the librairy with any function we will create as we are doing here, **but that codes duplication accross a lot of bunders**
> What we should do **it to save the request librairy in a separate storagesomewhere and the referenceit from the lambda that requires the library**,  this is a perfect use cases for the **lambda layer**
>
> A lambda layer **is a zip archive** that contain additionnal code or data that lambda can use.
>
> To create a layer:
1. Open a ``terminal``.
2. Create a folder name python(**it's a mandatoty**) and create a virtual environment .
3. Install the request libraries into the **python folder**
4. ````zip  -r request-layer.zip python````
5. Upload the layer from AWS console.
6. After uploading layer throught ``AWS Console`, don't forget to attch that layer to the function.`

**Note/**

When the lambda function is initialize, 
1. **The lambda service will unzip the layer and will place it in the** ```` /opt/python````
2. Lambda is running on the AMI Linux system

# C. Advanced Lambda concepts
## C.1 Triggers
**Triggers** are ressources that allows you to invoke lambda function reaction to an external event.

The **role** is basically said:
> **Me as lambda i can do this things**
>
The **resource-based policy**  saids:
> **Me as a lambda i can be invoke by these services**(generally a Principal)

## C.2 Cold & Warm start
When you invoke your lmabda for the first time, **the service need to prepare the execution environment** for your function.
1. The  services need to download the function code that is stored in an internel S3 bucket .
2. Then it has to create an environment with the specified ùmemory, the runtime ans all of other configurations.
3. Only after all theses resources are allocated to an instance of your function, **it can process the event**.
   
   > This initialization period is frequently referred to as a **cold start**.

   > If another event comes in right after the first one was processed, **the initialization period is skipped; because we have a lambda that's ready to be invoked**. This is called a **warm start**

   > If our lambda doesn't receive any events from some periods of time,  **the lambda service decodes to kill the lambda instance**  

   > The time between the **last invokation** and **the service deciding to kill the instance** is unknown and internal to ``AWS``.

   > Because of this concept of **cold start** and **warm start**, it **recommended to initialize some of your ressources outside the function handler**  when the function is invoked for the first time. 
   
   > **When a function is invoke at the first time i.e at the cold start, all the code outside the handler is run**. At **warm start** the code outside the handler are already been imported.

   > **Remember:** You need to initialize database connecxion outside the handler.

A **cold start** occurs when a **new execution environment is required to run a Lambda function**. When the Lambda service receives a request to run a function, the service first prepares an execution environment. During this step, the service downloads the code for the function, then creates the execution environment with the specified memory, runtime, and configuration. Once complete, Lambda runs any initialization code outside of the event handler before finally running the handler code. 

In a **warm start**, the **Lambda service retains the environment instead of destroying it immediately**. This allows the function **to run again within the same execution environment**. This saves time by not needing to initialize the environment.  
## C.3 Asynchronousand/Synchronous invocation
The are two ways lambda function can be invoke:
   * **Synchronous invocation**: Where the client send request to a lambda; the lambda does it job (executes the codes we've uploaded, computation, connecxion to a database and so on) and returns a response to the client. So as a **caller** you know whether or not that function was successful.

   * **Asynchronous invocation**: Where client doesn't sent a request to the lamnda, it places it on a queue that is internal to the lambda. After the client successfully places the event on the queue, it considers its job done, **the client is not waiting any response from the lambda**. When the lambda is not busy, it takes the request even from the sueue and processes it.
   * > **Here we follow the process of drop the message and move on.** We don't care wherher or not it actually succeeded.

When we click on a **test buttom** in the console, ware invoking **the functions synchronously**, we need to xait for lambda to return a response.
> **You have the choice of using Asynchronous or synchronous, this is control via a parameter that you pass in when you're using the SDK**
> The lambda function is not bound to the invokation type and you can switch between these two at any time**

## C.4 Dead Letter Queue
A **Dead Leter queue** is queue to which messages in our ceses events are sent if they cannot be successfull process by our lambda.
To create a  dead letter queue, we'll use ``AWS SQS``  and add it to our lambda function(go to configuration + Asynchronousand invocation+ Attach+ Dead Letter Queue)

## C.5 Concurrency
Thinks that happens at the same time.
> **Concurrency is the number of requests being served at a given moment**. It's something you need to think about when designing an application with Lambda.

> **Concurrency is a major scalling consideration, and can cause applications to fail dur to throttling**. Defaults is 1000 units of concurrency per AWS account.

The are three type of   concurrencies:
* **Unreserved** : The deafult one (common concurrency pool).
  * If one function consumes all consurrency, **the others will get throttled**.
  * Limit can be raised via AWS Support Ticket.
* **Reserved**   : Reserved a certain number of concurent units for your Lambda. This function will always have those X number of units concurrency(Dedicated concurrency pool).
  * A portion is always reserved for a specific function, even if there are no invocations.
  * You can use reserved concurrency to minimize or maximize your processing rate.
* **Provisioned**: Here you **have a dedicated ans always on concurrency pool that is specifeid at the function and the version level**.
> **It makes it so that you do,n't have a cold start problem**
  * Supports autoscalling policies based on usage.
  * If i said that for my function, i want to specify provision concurrency on it, and i can said that i always want to have 10 units of provision concurrency, **that means that Lambda will reserve and allocate ressources for 10 containers that are going to be able  to serve traffic without having to cold start**.
  * It's very, very expensive.
  * **Provision concurrency** substract from **reserved concurrency** and will rnever exceed the number of **reserved concurrency** that you've specified.

**Throttling** aka **RateExceeded**
* Throttling is when Lambda rejects a request
* Occurs when in flight invocations exceeds available concurrency.



**Pro Tips**
* Setup an alarm on throttles for early indicators of issues.
* Evaluate your concurrency needs and plan accordingly.
* Have your clients use Exponentiel Backoff to avoid retry storms.
* Raising your Memory Limit can help, **but be careful**.
* Use a small amount of provisioned concurrency to mitigate cols strats for latency sensitive apps.
* ## C.6 Execution environment lifecycle

![lambda](https://github.com/tkamag/aws-ressources/assets/14333637/00a15151-a98e-4895-a9a1-67a182ebb909)

When you create your Lambda function, you specify configuration information, such as the amount of available memory and the maximum invocation time allowed for your function. Lambda uses this information to set up the execution environment.

The function's runtime and each external extension are processes that run within the execution environment. Permissions, resources, credentials, and environment variables are shared between the function and the extensions.
### C.6.1 Init phase
In this phase, 
* Lambda creates or unfreezes an execution environment with the configured resources, 
* Downloads the code for the function and all layers, 
* Initializes any extensions, 
* Initializes the runtime, and then 
* Runs the **function’s initialization code (the code outside the main handler)**. 

> The Init phase happens **either during the first invocation**, or **before function invocations** if you have enabled provisioned concurrency.

The Init phase is split into three sub-phases: 

1. **Extension init** - starts all extensions
2. **Runtime init** - bootstraps the runtime
3. **Function init** - runs the function's static code

> These sub-phases **ensure that all extensions and the runtime complete their setup tasks before the function code runs**.
>
> ### C.6.2 Invoke phase
In this phase, 
* **Lambda invokes the function handler**. 

After the function runs to completion, Lambda prepares to handle another function invocation. 

### C.6.3 Shutdown phase
If the **Lambda function does not receive any invocations for a period of time, this phase initiates**. In the Shutdown phase,
* Lambda shuts down the runtime, 
* Alerts the extensions to let them stop cleanly, and then 
* Removes the environment. 

Lambda sends a shutdown event to each extension, which tells the extension that the environment is about to be shut down.

> When you write your function code, **do not assume that Lambda automatically reuses the execution environment for subsequent function invocations**. Other factors may require Lambda to create a new execution environment, which can lead to unexpected results. Always test to optimize the functions and adjust the settings to meet your needs.
>
> [Lambda execution environment](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtime-environment.html)
>
### C.7 Best practice: Minimize cold start times
When you invoke a Lambda function, **the invocation is routed to an execution environment to process the request**. 
* If the environment is not already initialized, **the start-up time of the environment adds to latency**. 
* If a function has not been used for some time, if more concurrent invocations are required, or if you update a function, new environments are created.  

Creation of these environments can introduce latency for the invocations that are routed to a new environment. This latency is implied when using the term **cold start**. For most applications, this additional latency is not a problem. However, for some synchronous models, this latency can inhibit optimal performance. 
> **It is critical to understand latency requirements and try to optimize your function for peak performance.** 

After optimizing your function, **another way to minimize cold starts is to use provisioned concurrency**. Provisioned concurrency is a Lambda feature that prepares concurrent execution environments before invocations.

### C.8 Versions
Versions are snapshots of the code and settings of a function.
* Code and settings get locked to a version on publish.
* Version is absolutely optionnal.
* New default default to **$LATEST** version.

### C.9 Alias
An alias is an **named pointer to a specific version**.
* Useful for beta testing or pre-prod environments.
* You can assing weights to different versions using a **Version/Alias**.
* To create an **Alias**, you first create a **version** and map this version to the **alias** and the **alais** pointed to **only one version** at a time.
* The top level function, the latest one is **$LATEST**

### C.10 Environment Variables
* Pairs of Strings with a key and value
* Allow you to adjust function behavior without touching your code.
* Examples:
  * STAGE
  * CLIENT_ENDPOINT_URL
  * DATABASE_CONNNECTION_STRING

````python
import os

db_connection_string = os.environ['DATABASE_CONNNECTION_STRING']`
region = os.environ['AWS_REGION']

````



## D. AWS Lambda Function Permissions
With Lambda functions, there are two sides that define the necessary scope of permissions – 
* **Permission to invoke the function**, who are controlled using an **IAM resource-based policy**
* **Permission of the Lambda function itself to act upon other services**. An **IAM execution role** defines the permissions that control what the function is allowed to do when interacting with other AWS services. 

![iam](https://github.com/tkamag/SageMaker/assets/14333637/3487ea06-0e5b-429c-b6c0-d76f7681a198)

> **Resource policies** grant permissions to invoke the function, whereas the **execution role** strictly controls what the function can to do within the other AWS service.
>
> **Remember to use the principle of least privilege when creating IAM policies and roles**. Always start with the most restrictive set of permissions and only grant further permissions as required for the function to run. Using the principle of least privilege ensures security in depth and eliminates the need to remember to 'go back and fix it' once the function is in production.
### D.1 Execution role definitions
> 
![iam](https://github.com/tkamag/aws-ressources/assets/14333637/5546e5c1-e79c-4bcc-b353-07853cee487b)

![trust](https://github.com/tkamag/aws-ressources/assets/14333637/198a0792-807a-4a06-96bc-26fa5e40bdd7)

### D.2 Resource-based policy
A **resource policy** (also called a function policy or IAM ressource policy) **tells the Lambda service which principals have permission to invoke the Lambda function**. 
> An **AWS principal may be a user, role, another AWS service, or another AWS account**.
>
The resource-based policy is an easier option and you can see and modify it via the Lambda console. A consideration with cross-account permissions is that a **resource policy does have a size limit**. If you have many different accounts that need to invoke the function and you have to add permissions for each account via the resource policy, you might reach the policy size limit. In that case, **you would need to use IAM roles instead of resource policies**. 

The following is a basic resource policy example.

* The policy has an Effect of ``Allow``. The Effect can be Deny or Allow.
* The Principal is the Amazon S3 ``s3.amazonaws.com`` service. This **policy is allowing the Amazon S3 service to perform an Action**.
* The Action that **S3 is allowed to perform is the ability to invoke a Lambda function** ``lambda:InvokeFunction`` called ``my-s3-function``.* 

![call](https://github.com/tkamag/aws-ressources/assets/14333637/c6d9deda-edbd-4b10-bce9-f5d5a4d0aae7)

### D.3 Policy comparison
| **Ressiurce-based policy**  | **Execution role**  |
|---|---|
| Lambda resource-based (function) policy  | IAM execution role  |
| * Associated with a "push" event source such as Amazon API Gateway  | * Role selected or created when you create a Lambda function  |
| * Created when you add a trigger to a Lambda function | * IAM policy includes actions you can take with the resource  |
| * Allows the event source to take the lambda:InvokeFunction action  |  * Trust policy that allows Lambda to AssumeRole |
|   | * Creator must have permission for iam:PassRole  |

## E. Ease of management

For ease of policy management, you can use authoring tools such as the ``AWS Serverless Application Model (AWS SAM)`` to help manage your policies. For a Lambda function, ``AWS SAM`` **scopes the permissions of your Lambda functions to the resources used by your application**. You can add IAM policies as part of the ``AWS SAM`` template. The policies property can be the name of AWS managed policies, inline IAM policy documents, or AWS SAM policy templates.

## F. Accessing resources in a VPC

Enabling your Lambda function to access resources inside your virtual private cloud (VPC) requires additional VPC-specific configuration information, such as VPC subnet IDs and security group IDs. This functionality allows Lambda to access resources in the VPC. It does not change how the function is secured. You also need an execution role with permissions to create, describe, and delete elastic network interfaces. Lambda provides a permissions policy for this purpose named ``AWSLambdaVPCAccessExecutionRole``.

### F.1 Lambda and AWS PrivateLink

To **establish a private connection between your VPC and Lambda, create an interface VPC endpoint**. 
> **Interface endpoints** are powered by ``AWS PrivateLink,`` which enables you to privately access Lambda APIs without an internet gateway, NAT device, VPN connection, or AWS Direct Connect connection. 
>
Instances in your VPC don't need public IP addresses to communicate with Lambda APIs. Traffic between your VPC and Lambda does not leave the AWS network. 

**Remember**

A **resource policy determines who is allowed in (who can initiate your function, such as Amazon S3)**, and it **can be used to grant access across accounts**. 
* Can give Amazon S3 permission to initiate a Lambda function
* Can grant access to the Lambda function across AWS accounts
* Determines who has access to invoke the function

An **execution role must be created or selected when creating your function**, and **it controls what Lambda is allowed to do (such as writing to a DynamoDB table)**. It includes a **trust policy** with **AssumeRole**. * Must be chosen or created when you create a Lambda function
* Can give Lambda permission to write data to a DynamoDB table
* Determines what Lambda is allowed to do

**Even source mapping** : Fancing terms who said that, instead of invokink my function, **lambda is going to read event from a queue (SQS or SNS or DynamoDB Streams)  and invoke your function as a result of seeing messages in that queue**

## G. Metrics to Monitor
* **Invocations**
* **Errors count and success rate**
* **Duration**
* **Throtttle**
* **ConcurrentExecutions**
* **UnreserveredConcurrentExecutions**
* **IteratorAge**: For **DynamoDB stream** or **Kinesis** and it's the difference in time between when a message was put into the stream and when the message is pull of the stream.
  * When the time is large, then Lambda is not processing enough
  * When it's near zero, it's a good thing.

**Pro Tips**
* Metrics don't help if you're not lookng at them.
* SetUp alarm on your metrics.