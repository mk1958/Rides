# AWS Project - Build a Full End-to-End Web Application with 7 Services | Step-by-Step Tutorial

This repo contains the code files 

## TL;DR
We're creating a web application for a unicorn ride-sharing service called Wild Rydes (from the original [Amazon workshop](https://aws.amazon.com/serverless-workshops)).  The app uses IAM, Amplify, Cognito, Lambda, API Gateway and DynamoDB, with code stored in GitHub and incorporated into a CI/CD pipeline with Amplify.

The app will let you create an account and log in, then request a ride by clicking on a map (powered by ArcGIS).  The code can also be extended to build out more functionality.

## Cost
All services used are eligible for the [AWS Free Tier](https://aws.amazon.com/free/).  Outside of the Free Tier, there may be small charges associated with building the app (less than $1 USD), but charges will continue to incur if you leave the app running.  Please see the end of the YouTube video for instructions on how to delete all resources used in the video.

## The Application Code
The application code is here in this repository.

## The Lambda Function Code
Here is the code for the Lambda function, originally taken from the [AWS workshop](https://aws.amazon.com/getting-started/hands-on/build-serverless-web-app-lambda-apigateway-s3-dynamodb-cognito/module-3/ ), and updated for Node 20.x:


🚀 AWS Serverless Project: Ride-Sharing Web Application
📌 Project Overview

This project demonstrates the design and implementation of a scalable, serverless ride-sharing web application using core AWS services. The application allows users to sign up, log in, and request rides through an interactive interface.

It follows a modern cloud-native architecture, eliminating the need for server management while ensuring high availability, security, and scalability.

🧩 AWS Services Used

1) AWS Amplify – Frontend hosting & CI/CD
2) Amazon Cognito – User authentication (signup/login)
3) AWS Lambda – Backend logic execution
4) Amazon API Gateway – API exposure
5) Amazon DynamoDB – Data storage
6) AWS IAM – Access control
 
 
 🏗️ Architecture Overview
User → Frontend (Amplify)
     → Cognito (Authentication)
     → API Gateway
     → Lambda Function
     → DynamoDB


⚙️ Step-by-Step Implementation
🔹 Step 1: Frontend Deployment (Amplify)
Upload your frontend code (HTML, CSS, JS) to GitHub
Connect repository to AWS Amplify
Enable automatic deployment (CI/CD)

👉 Result:

Your web app is hosted and accessible via a public URL

🔹 Step 2: User Authentication (Cognito)
Create a User Pool
Enable:
Email-based signup/login
OTP verification
Create an App Client

👉 Flow:

User → Signup (Email)
     → Verify OTP
     → Login
     → Receive JWT Token

     
🔹 Step 3: Create DynamoDB Table
Table Name: Rides
Primary Key: RideId (String)

👉 Stores:

Ride ID
Username
Ride details
Timestamp


🔹 Step 4: Backend Logic (Lambda)
📌 Purpose

Handles ride requests and stores them in DynamoDB.

🔹 Key Responsibilities
Validate user authentication
Generate unique ride ID
Assign a ride (unicorn/vehicle)
Store ride data

🔹 Step 5: API Gateway Integration
Create REST API
Add POST endpoint: /ride
Connect to Lambda
Enable:
CORS
Cognito Authorizer

👉 Flow:

Frontend → API Gateway → Lambda → DynamoDB
🔹 Step 6: Secure API with Cognito
Configure Cognito Authorizer
Attach to /ride endpoint

👉 Ensures:

Only authenticated users can access API


🔹 Step 7: IAM Role Configuration
Create role for Lambda
Attach permissions:
DynamoDB access



Step 8: Testing the Application
Sample Request:
{
  "PickupLocation": {
    "Latitude": 47.6174,
    "Longitude": -122.2883
  }
}
Expected Response:
{
  "RideId": "abc123",
  "Unicorn": {...},
  "Eta": "30 seconds",
  "Rider": "username"
}


```node
import { randomBytes } from 'crypto';
import { DynamoDBClient } from '@aws-sdk/client-dynamodb';
import { DynamoDBDocumentClient, PutCommand } from '@aws-sdk/lib-dynamodb';

const client = new DynamoDBClient({});
const ddb = DynamoDBDocumentClient.from(client);

const fleet = [
    { Name: 'Angel', Color: 'White', Gender: 'Female' },
    { Name: 'Gil', Color: 'White', Gender: 'Male' },
    { Name: 'Rocinante', Color: 'Yellow', Gender: 'Female' },
];

export const handler = async (event, context) => {
    if (!event.requestContext.authorizer) {
        return errorResponse('Authorization not configured', context.awsRequestId);
    }

    const rideId = toUrlString(randomBytes(16));
    console.log('Received event (', rideId, '): ', event);

    const username = event.requestContext.authorizer.claims['cognito:username'];
    const requestBody = JSON.parse(event.body);
    const pickupLocation = requestBody.PickupLocation;

    const unicorn = findUnicorn(pickupLocation);

    try {
        await recordRide(rideId, username, unicorn);
        return {
            statusCode: 201,
            body: JSON.stringify({
                RideId: rideId,
                Unicorn: unicorn,
                Eta: '30 seconds',
                Rider: username,
            }),
            headers: {
                'Access-Control-Allow-Origin': '*',
            },
        };
    } catch (err) {
        console.error(err);
        return errorResponse(err.message, context.awsRequestId);
    }
};

function findUnicorn(pickupLocation) {
    console.log('Finding unicorn for ', pickupLocation.Latitude, ', ', pickupLocation.Longitude);
    return fleet[Math.floor(Math.random() * fleet.length)];
}

async function recordRide(rideId, username, unicorn) {
    const params = {
        TableName: 'Rides',
        Item: {
            RideId: rideId,
            User: username,
            Unicorn: unicorn,
            RequestTime: new Date().toISOString(),
        },
    };
    await ddb.send(new PutCommand(params));
}

function toUrlString(buffer) {
    return buffer.toString('base64')
        .replace(/\+/g, '-')
        .replace(/\//g, '_')
        .replace(/=/g, '');
}

function errorResponse(errorMessage, awsRequestId) {
    return {
        statusCode: 500,
        body: JSON.stringify({
            Error: errorMessage,
            Reference: awsRequestId,
        }),
        headers: {
            'Access-Control-Allow-Origin': '*',
        },
    };
}
```

## The Lambda Function Test Function
Here is the code used to test the Lambda function:

```json
{
    "path": "/ride",
    "httpMethod": "POST",
    "headers": {
        "Accept": "*/*",
        "Authorization": "eyJraWQiOiJLTzRVMWZs",
        "content-type": "application/json; charset=UTF-8"
    },
    "queryStringParameters": null,
    "pathParameters": null,
    "requestContext": {
        "authorizer": {
            "claims": {
                "cognito:username": "the_username"
            }
        }
    },
    "body": "{\"PickupLocation\":{\"Latitude\":47.6174755835663,\"Longitude\":-122.28837066650185}}"
}
```

