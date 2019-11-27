# Abstract
Handling data using the RESTful architecture can become challenging in big applications. This is because, in RESTful, the client must work with many endpoints with a predefined data structure. That typically leads to over fetching or under fetching data, which of course affect the performance.  <br/>
Many endpoints mean many network calls and that leads to bad performance. Likewise, predefined data structures mean getting undesired data or missing desired data and that leads to bad performance. And no doubt, performance means a lot, especially for applications with many users and many integrations to 3rd party software.  <br/>
GraphQL is a language, which solves many challenges of the RESTful architecture.  
Firstly, it exposes only one endpoint for all entities.
Secondly, the data and its structure are not predefined, rather the client defines what data it wants and how the structure should be.  <br/>
Using this architecture, we achieve better performance due to the reduced number of network calls. And also, we ourselves specify the data we need, which reduces the unnecessary data transferring.
 
# Introduction
GraphQL is a query language for server-side runtime and query execution. GraphQL is a new API standard that was developed and open sourced by Facebook.  
It is not a specific technology but can implement it in any language.
In GraphQL we have four core components, namely the `Schema`, `Query` and `Mutation`.  
The `Schema` is at the core of any GraphQL server implementation and it is language independent. It describes the functionality available to the client applications that connect to it.  
The `Query` is a `type` which is used for the `R` in `CRUD`.
Likewise, `Mutation` is also a `type` which is used for the `CUD` in `CRUD`.  
Hence, the `Query` can roughly be compared to `GET` and `Mutation` can roughly be compared to `POST`, `PUT`, `PATCH` and `DELETE` of the RESTful approach.  
However, you will soon see, that we only have one endpoint for both `Query` and `Mutation` operations in GraphQL.  

> In this blog, we will be comparing the GraphQL approach with the RESTful approach and understanding the pros of GraphQL.
More precisely, we will look at the differences between the data structure and the exposure of endpoints in both GraphQL and RESTful.
 
# The Driving School Software
Some time ago, I had to build a system for driving schools which should be accessible from both desktop and mobile devices. The concept was to make schools able to organise their teachers, students, teams, lectures and other data.  
 
We realised that the system might become very complex, therefore we chose to use GraphQL rather than the RESTful approach.  
 
The main idea of GraphQL is, as mentioned earlier, that we only have one endpoint for all our entities. Moreover, the result data is defined by the client - according to the `Schema` mentioned earlier - not the server.  
 
First and foremost, let's look at the difference between the structure of endpoints for both RESTful and GraphQL.  
 
For example, for the above-mentioned system the structure of the endpoints with RESTFul could have been something like:
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
On the other hand, in GraphQL, we only have one single endpoint. It could possibly be:
`GET: /graphql?query=:query`
Where `:query` is the actual query we want to process.  
 
Of course, in the backend we have to make the endpoint available for the client. In our case, the schema with `student` and `teacher` for `Query` would be something like:
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
Here, we only describe the kind of data that is available. And the structure doesn’t tell us anything about how a client might fetch  those objects from our server.  
 
Likewise, we will define the `Mutation` schema for `student`:
```graphql
type Mutation {
  addStudent(id: Int, firstNmae: String, lastName: String): String
}
```
Here, we define the schema for adding a student to the system.  
 
In this blog, we will base our future examples on the schemas just defined.  
 
Now, let's look at how we can retrieve the first name of a specific student with the first name of his teachers with a RESTful approach:
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
Now we have to get the student’s teachers with their first names:
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
Firstly, as we just observed, we have to make at least two requests to the server.  
Secondly, we are actually getting data we didn't intend to request. Of course, we could make a parameter which accepts fields, such as, `GET: /students/5?fields=firstName` but that will quickly become complex and difficult to manage in bigger applications.  
 
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
Here, we only called the server once and the server only returned the data we intended to request. That of course has an effect on the performance.  
 
 
Now, let's have a look at how we can write data using a RESTful approach. Firstly, we need to call the endpoint using `POST` - of course we also have to send a payload:
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
We have created the student but the relation between the student and his teachers is not defined yet. To define the relation we can do something like:
```
POST /students/423/teachers HTTP/1.1
HOST: myserver
Content-Type:application/json
Accept:application/json
{
  "data": {
    "teachersID": [8, 2, 3]
}
```
This will create a relation between the newly created student and his teachers.  
 
The very same thing can be done seamlessly using GraphQL:
```graph
mutation {
  addStudent(id: 423, firstName: "Jack", lastName: "Green", teachers: [8, 2, 3]) {
    firstName
  }
```
This single request to the server ensures that our new student is created in the system and he is associated - in the system - with his teachers correctly.  
 
As we can see, the RESTful approach can quickly become complex, where as the GraphQL approach will remain with a simple and clear structure.  
 
The following image, which does not have anything to do with the Driving School Software, illustrates the interpretation of fetching resources with multiple REST roundtrips vs. one single GraphQL request.
![GraphQL vs RESTful](images/graphql-restful.png)  
*Image source: https://blog.apollographql.com/graphql-vs-rest-5d425123e34b*
 
 
# Conclusion:
We would not say that GraphQL is a replacement of the RESTful approach but clearly it has some features that RESTful doesn't have. It makes joins, filtering, argument validation etc. much easier compared to RESTful.  
 
We can summarise the similarities and differences between the two in the following points:
## Similarities
 * Both have the idea of a resource, and can specify IDs for those resources.
 * Both can return JSON data in the request.
 * Both have a way to differentiate if an API request is meant to read data or write it.
 
## Differences
 * In REST, the shape and size of the resource is determined by the server. In GraphQL, the server declares what resources are available, and the client asks for what it needs at the time.
 * In GraphQL, you can traverse from the entry point to related data, following relationships defined in the schema, in a single request. In REST, you have to call multiple endpoints to fetch related resources.
 * In REST, you specify a write by changing the HTTP verb from GET to something else like POST. In GraphQL, you change the `query` keyword to `mutation` in the query.