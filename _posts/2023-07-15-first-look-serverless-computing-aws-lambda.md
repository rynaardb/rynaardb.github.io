---
layout: post
title: "A First Look at Serverless Computing - Exploring AWS Lambda"
date: 2023-07-15
categories: serverless aws
tags: serverless aws lambda
image:
    path: /assets/images/aws-lambda.webp
mermaid: true
---

In recent years, serverless computing has emerged as a revolutionary concept that's shaping the landscape of cloud computing. By providing developers the freedom to focus solely on their code, and liberating them from server management or capacity planning, it's significantly streamlining the application development process.

Let’s look into some of the concepts of serverless computing, how it works, its pros and cons, and then a practical example using AWS Lambda.

## What is Serverless Computing?

Contrary to the term, serverless computing doesn't mean computing without servers. Rather, it refers to *a model where developers can build and run applications without having to worry about the underlying infrastructure*. In the serverless model, the cloud provider manages the servers and dynamically allocates the resources as per the workload requirements.

With serverless computing, the infrastructure scales up and down automatically, and you only pay for the compute time you consume. This eliminates the need for server provisioning and maintenance, resulting in faster development cycles and lower costs.

## Serverless Architecture

Let's dive a little deeper into the serverless architecture and understand its key components.

### Function as a Service (FaaS)

FaaS is the cornerstone of the serverless architecture. It lets you execute a piece of code (a function) in response to events (like a file upload to a cloud storage bucket, an API gateway call, or a change in a database table). The FaaS platform manages all the physical resources, and developers only have to focus on writing the code for the function.

### Backend as a Service (BaaS)

In serverless architectures, third-party services (BaaS) are often used to manage server-side logic and state. These could be databases (like DynamoDB or Firestore), authentication services (like AWS Cognito or Firebase Authentication), or storage services (like S3 or Google Cloud Storage).

### Event Sources

Event sources are the triggers for the functions in a serverless architecture. An event source can be a HTTP request, a modification in a database, or a system status change, among others.

## Pros and Cons of Serverless Architecture

Like any technology, serverless architecture has its benefits and drawbacks.

### Pros

#### Scalability

Serverless applications can scale automatically based on the workload. This means no idle resources and cost savings for your business.

#### Productivity

With the infrastructure managed by the provider, developers can focus solely on writing the code, leading to improved productivity.

#### Cost-Effective

You only pay for the compute time you consume, making serverless architecture a cost-effective choice for small to large scale applications.

### Cons

#### Cold Start

The first request to a serverless function can experience a delay known as a cold start. This delay can impact user experience, especially for real-time applications.

#### Debugging and Monitoring

Debugging serverless applications can be challenging due to the distributed nature of the system. Moreover, monitoring these applications requires specific tools and practices.

#### Vendor Lock-In

Applications built on serverless architectures can be highly dependent on the cloud provider's capabilities and services, making migration challenging....

## Creating a Serverless API with AWS Lambda and API Gateway

Let's walk through an example of creating a serverless API using AWS Lambda and API Gateway. In this example, we'll create an API that retrieves user data from a mock database.

First, you need to install the AWS CLI and AWS SDK for your preferred language. You can find the installation instructions in the AWS Documentation.

### Create a Lambda Function

We'll start by creating a Lambda function in Go:

```go
package main

import (
  "encoding/json"
  "fmt"
  "github.com/aws/aws-lambda-go/events"
  "github.com/aws/aws-lambda-go/lambda"
  "net/http"
  "strconv"
)

type User struct {
  Name string
  Age  int
}

func handleRequest(request events.APIGatewayProxyRequest) (events.APIGatewayProxyResponse, error) {
  // Mock response
  userData := map[int]User{
    123: {Name: "Alice", Age: 25},
    456: {Name: "Bob", Age: 32},
    789: {Name: "Charlie", Age: 29},
  }

  userId, err := strconv.Atoi(request.PathParameters["userId"])
  if err != nil {
    return events.APIGatewayProxyResponse{}, err
  }

  if user, ok := userData[userId]; ok {
    userJson, _ := json.Marshal(user)
    return events.APIGatewayProxyResponse{
      Body:       string(userJson),
      StatusCode: http.StatusOK,
    }, nil
  } else {
    return events.APIGatewayProxyResponse{
      Body:       fmt.Sprintf(`{"error": "User %d not found"}`, userId),
      StatusCode: http.StatusNotFound,
    }, nil
  }
}

func main() {
  lambda.Start(handleRequest)
}
```
{: file="main.go" }

This function accepts a `userId` and returns the corresponding mock user data. If the user is not found, it returns a 404 error.

We use the `github.com/aws/aws-lambda-go/events` and `github.com/aws/aws-lambda-go/lambda` packages, which provide types and functions for working with AWS Lambda and API Gateway events.

The `handleRequest` function takes an `events.APIGatewayProxyRequest` and returns an `events.APIGatewayProxyResponse`. The user data is stored in a map, and the `userId` from the request parameters is used to look up the user. If the user is found, the function returns a response with the user's data; otherwise, it returns a "User not found" error.

Finally, the `main` function starts the Lambda function by calling `lambda.Start` with our `handleRequest` function.

### Deploy the Lambda Function

To deploy this function, navigate to the AWS Lambda console, click "Create Function", and follow the instructions. You can name the function `getUserData`.

### Create an API Gateway

Now, let's create an API Gateway that triggers our Lambda function. Navigate to the API Gateway console, click "Create API", select "REST API", and follow the instructions. You can name the API `userDataAPI`.

After the API is created, you need to create a resource and a GET method. For the resource name, enter `user`, and for the resource path, enter `{userId}`. This allows us to pass the `userId` as a parameter in the URL.

For the GET method, select the `getUserData` Lambda function as the integration point.

### Deploy the API Gateway

Finally, deploy the API Gateway by clicking "Actions" and then "Deploy API". You need to create a new deployment stage. You can name it `dev`.

Once the API is deployed, you will get an invoke URL. You can use this URL to test the API:

`https://<api-id>.execute-api.<region>.amazonaws.com/dev/user/123`

>Replace `<api-id>` with your API ID and `<region>` with your AWS region. The final part of the URL (`/user/123`) is the path to the resource and the `userId`.
{: .prompt-info }

## Conclusion

Serverless computing has the potential to transform how we build and deploy applications in the cloud. By handling server management and capacity planning, serverless architectures allow developers to focus on writing code, resulting in faster delivery times and lower operational costs.

However, serverless is not a one-size-fits-all solution, and it's essential to consider its potential drawbacks, like cold starts and vendor lock-in, when designing your systems.

There's much more to explore with serverless technologies, including integration with databases, data processing pipelines, and machine learning models. The sky's the limit when it comes to what you can build with serverless!
