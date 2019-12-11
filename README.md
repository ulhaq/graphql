# A look into GraphQL
Authors:  
Muhammad Umar Ulhaq (cph-mu47@cphbusiness.dk)  
Hassan Raza Hussain (cph-hh266@cphbusiness.dk)  
Mohammed Murad Hossain Sarker (cph-ms809@cphbusiness.dk) 
 
10th December 2019

# Abstract
Handling data using the RESTful architecture can become challenging in big applications. This is because the fact, that with a RESTful approach, the client must work with many endpoints with a predefined data structure.  
<br/>
Many endpoints lead to many network calls. Likewise, predefined data structure typically leads to over fetching or under fetching data.  
<br/>
GraphQL is a language, which solves many challenges of the RESTful architecture.
Firstly, it exposes only one endpoint for all resources. Secondly, the data and its structure are not predefined, rather the client defines what data it wants and how the structure should be.  
<br/>
Using this architecture we achieve better overview of our endpoint - we only have one endpoint. And, we ourselves specify the data we need, which reduces the transferring of unnecessary data.  

# Introduction
GraphQL is a query language for server-side runtime and query execution. GraphQL is a new API standard which was developed and open sourced by Facebook.  
We will be looking at three of GraphQL's main core components, namely the `Schema`, `Query` and `Mutation`.  
The `Schema` describes the functionality available to the client applications that connect to it.  
The `Query` is a `type` which is used for the `R` in `CRUD`.  
The `Mutation` is also a `type` which is used for the `CUD` in `CRUD`.  

Hence, the `Query` can roughly be compared to GET and `Mutation` can roughly be compared to `POST`, `PUT`, `PATCH` and `DELETE` of the RESTful approach.  

You will soon see that we only have one endpoint for both `Query` and `Mutation` operations in GraphQL.
<br/>
> **In this blog, we will compare the GraphQL approach with the RESTful approach and understanding the pros of GraphQL.
More precisely, we will look at the differences between the data structure and the exposure of endpoints in both GraphQL and RESTful.**

# The Driving School Software
Some time ago, we had to build a system for driving schools which should be accessible from both desktop and mobile devices. The concept was to make schools able to organise their teachers, students, teams, lectures and other data.  

We realised that the system might become very complex, therefore we chose to use GraphQL rather than the RESTful approach.  

The main idea of GraphQL is that we only have one endpoint for all our resources. And the result data is defined by the client – in accordance with the `Schema` mentioned earlier - not the server.  

First and foremost, let's look at the difference between the structure of endpoints for both RESTful and GraphQL.  

For example, for the Driving System the structure of some endpoints with RESTful could have been something like:
```
GET: /teachers/
GET: /teachers/:id
GET: /teachers/:id/students
POST: /teachers/
PUT: /teachers/:id
DELETE: /teachers/:id

GET: /students/
GET: /students/:id
GET: /students/:id/teachers
POST: /students/
PUT: /students/:id
DELETE: /students/:id
```
On the other hand, in GraphQL, we only have one single endpoint. It could possibly be: `GET: /graphql?query=:query` where `:query` is the actual query we want to process.  

Of course, in the backend, we have to code the business logic and make the endpoint available for the client.  
However, the business logic part is out of the scope of this blog, since we are only focusing on endpoints and data structure.<sup><a href="#fn1" id="fnref1">[1]</a></sup>  

In our case, the schema with `student` and `teacher` for `Query` would be something like:
```graphql
type Student {
    id: ID
    firstName: String
    lastName: String
    teachers: [Teacher]
}

type Teacher {
    id: ID
    firstName: String
    lastName: String
    students: [Student]
}
```
Here, we only describe the kind of data that is available to the client. And the structure doesn’t tell us anything about how a client might fetch those objects from our server.  

Likewise, we can define the `Mutation` schema for `student`:
```graphql
type Mutation {
    addStudent(id: Int, firstNmae: String, lastName: String): String
}
```
Here, we define the schema for adding a student to the system.  

In this blog, we will base our future examples on the schemas just defined.  

Let's look at how we can retrieve the first name of a specific student and the first name of his teachers with a RESTful approach:
```
GET: /students/5/
```
This will result in something like:
```json
{
    "data": {
        "students": {
            "id": 5,
            "firstName": "John",
            "lastName": "Doe"
        }
    }
}
```
We must get the student’s teachers' first names:
```
GET: /students/5/teachers/
```
This will result in something like:
```json
{
    "data": {
        "teachers": [
            {
                "firstName": "Muhammad",
                "lastName": "Ibraheem"
            },
            {
                "firstName": "Jane",
                "lastName": "Doe"
            },
            {
                "firstName": "Michael",
                "lastName": "Jeffrey"
            }
        ]
    }
}
```
Firstly, as we just observed, we must make at least two requests to the server – at least if we want to follow best practices. Secondly, we are actually getting data we didn't intend. Of course, we could make a parameter which accepts fields, such as, `GET: /students/5?fields=firstName` but that will quickly become complex and difficult to manage in bigger applications.  

However, the same thing can be achieved using GraphQL as follows:
```graphql
query {
    Student (id: 5) {
        firstName
        teachers () {
            firstName
        }
    }
}
```
This will result in something like:
```json
{
    "data": {
        "Student": {
            "firstName": "John",
            "teachers": [
                { "firstName": "Jane" },
                { "firstName": "Michael" },
                { "firstName": "Ibraheem" }
            ]
        }
    }
}
```
With this approach, we only called the server once and the server only returned the data we intended, namely the user's first name and his teachers' first name.  

Let's have a look at how we can write data using a RESTful approach. Firstly, we need to call the endpoint using `POST` - of course we must also send a payload:
```
POST /students/ HTTP/1.1
HOST: myserver
Content-Type:application/json
Accept:application/json
{
    "data": {
        "id": 423,
        "firstName": "Jack",
        "lastName": "Green",
    }
}
```
We have created the student but the relation between the student and his teachers is not defined yet.  
To define the relation, we can send a new request; that could be something like:
```
POST /students/423/teachers HTTP/1.1
HOST: myserver
Content-Type:application/json
Accept:application/json
{
    "data": {
        "teachersID": [8, 2, 3]
    }
}
```
This will create a relation between the newly created student and his teachers.  

The same thing can be done seamlessly using GraphQL:
```graph
mutation {
    addStudent(id: 423, firstName: "Jack", lastName: "Green", teachers: [8, 2, 3]) {
        firstName
    }
}
```
This single request, to the server, ensures that our new student is created in the system and he is associated to his teachers correctly.  

Hopefully you got the idea. The RESTful approach would become more complex in larger systems, whereas the GraphQL approach remains with a simple and clear structure.
Due to the length-limitation of this blog post, we didn't present too complex examples, even though that might help in making the idea clearer.  

The following image, which does not have anything to do with the Driving School Software, illustrates the interpretation of fetching resources with multiple REST roundtrips vs. one single GraphQL request.

![GraphQL vs RESTful](images/graphql-restful.png)  
*Image source: https://blog.apollographql.com/graphql-vs-rest-5d425123e34b*

# Conclusion
We would not say that GraphQL is a replacement of the RESTful approach. Nevertheless, it clearly has some features that RESTful doesn't have.  
Firstly, GraphQL exposes only one single endpoint and that gives a better overview and also reduces the time spent on designing multiple endpoints.  
Secondly, you are able to define the data structure on the client side. And that means that the server must only define the resources that are available to the client, not the data's structure. Hence, you can control what exactly data you want and avoid over fetching and under fetching of data. Imagine a system which communicates with desktop devices and mobile devices, where the desktop devices require data that mobile devices don't need and vice versa. That would lead to transferring useless data on different devices which would lead to increased internet bills for the end user.  
Thirdly, it makes things such as filtering much easier compared to RESTful.  

# Reference List
We have used the following resources for this blog post:

* https://graphql.org/learn/schema/
* https://graphql.org/learn/queries/
* https://graphql.org/learn/serving-over-http/
* https://www.howtographql.com/graphql-js/3-a-simple-mutation/
* https://medium.com/@camachojuan_18475/graphql-solving-rests-problems-9a78820aeff
* https://blog.apollographql.com/graphql-vs-rest-5d425123e34b
* https://www.tutorialspoint.com/graphql/graphql_schema.htm
* https://flaviocopes.com/graphql-node-express/  

# Footnotes
1. <span id="fn1">If you want to know more about implementation of GraphQL business logic have a look at the following <a href="https://flaviocopes.com/graphql-node-express/" target="_blank">blog post.</a></span>