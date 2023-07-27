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
``````python
return {
    'statuscode': 200,
    'body' : null
}
``````
