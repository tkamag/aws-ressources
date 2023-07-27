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
**Triggers** are ressources that allows you to invoke lambda function reaction to an external event.

The **role** is basically said:
> **Me as lambda i can do this things**
>
The **resource-based policy**  saids:
> **Me as a lambda i can be invoke by these services**(generally a Principal)
> 