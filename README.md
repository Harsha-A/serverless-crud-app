# serverless-crud-app
Serverless CRUD App with DynamoDB


To define a Serverless Framework `serverless.yml` file for a Lambda function that performs CRUD (Create, Read, Update, Delete) operations on an AWS DynamoDB table named `EmployeeTable`, you can use the following example. This assumes you're working with a Node.js runtime:

```yaml
service: employee-service
frameworkVersion: '2'

provider:
  name: aws
  runtime: nodejs14.x
  region: us-east-1 # Change to your desired AWS region

functions:
  createEmployee:
    handler: handler.createEmployee
    events:
      - http:
          path: /employees
          method: POST

  getEmployee:
    handler: handler.getEmployee
    events:
      - http:
          path: /employees/{id}
          method: GET

  updateEmployee:
    handler: handler.updateEmployee
    events:
      - http:
          path: /employees/{id}
          method: PUT

  deleteEmployee:
    handler: handler.deleteEmployee
    events:
      - http:
          path: /employees/{id}
          method: DELETE

custom:
  dynamodbTableName: EmployeeTable

resources:
  Resources:
    EmployeeTable:
      Type: AWS::DynamoDB::Table
      Properties:
        TableName: ${self:custom.dynamodbTableName}
        AttributeDefinitions:
          - AttributeName: id
            AttributeType: S
        KeySchema:
          - AttributeName: id
            KeyType: HASH
        ProvisionedThroughput:
          ReadCapacityUnits: 5
          WriteCapacityUnits: 5
```

Explanation:

- `service`: The name of your Serverless service.
- `frameworkVersion`: Specifies the Serverless Framework version.
- `provider`: AWS provider configuration, including the runtime and region.
- `functions`: Defines four Lambda functions for CRUD operations: `createEmployee`, `getEmployee`, `updateEmployee`, and `deleteEmployee`.
- `custom`: Custom variables to specify the DynamoDB table name.
- `resources`: Defines the AWS CloudFormation resources. In this case, it creates the `EmployeeTable` DynamoDB table.

Make sure to replace the handler function names (e.g., `handler.createEmployee`) with your actual Lambda function names. Additionally, ensure that you have the necessary code for these Lambda functions in your project.

After setting up your `serverless.yml`, you can deploy your service using the following command:

```bash
serverless deploy
```

This will create the Lambda functions and the `EmployeeTable` DynamoDB table in your AWS account. You can then use the specified HTTP endpoints to interact with the Lambda functions for CRUD operations on the DynamoDB table.






==================

Certainly! Below is a basic example of Node.js code for performing CRUD (Create, Read, Update, Delete) operations on an AWS DynamoDB table named `EmployeeTable` using the AWS SDK.

First, you'll need to install the AWS SDK for Node.js:

```bash
npm install aws-sdk
```

Next, you can create a Node.js script to interact with DynamoDB:

```javascript
const AWS = require('aws-sdk');

// Configure the AWS SDK with your AWS credentials and region
AWS.config.update({
  accessKeyId: 'YOUR_ACCESS_KEY',
  secretAccessKey: 'YOUR_SECRET_ACCESS_KEY',
  region: 'us-east-1', // Replace with your desired AWS region
});

const dynamodb = new AWS.DynamoDB.DocumentClient();

const tableName = 'EmployeeTable';

// Function to create a new employee record
const createEmployee = (employee) => {
  const params = {
    TableName: tableName,
    Item: employee,
  };

  return dynamodb.put(params).promise();
};

// Function to get an employee by their ID
const getEmployee = (employeeId) => {
  const params = {
    TableName: tableName,
    Key: {
      id: employeeId,
    },
  };

  return dynamodb.get(params).promise();
};

// Function to update an employee's information
const updateEmployee = (employee) => {
  const params = {
    TableName: tableName,
    Key: {
      id: employee.id,
    },
    UpdateExpression: 'set firstName = :firstName, lastName = :lastName',
    ExpressionAttributeValues: {
      ':firstName': employee.firstName,
      ':lastName': employee.lastName,
    },
    ReturnValues: 'ALL_NEW',
  };

  return dynamodb.update(params).promise();
};

// Function to delete an employee by their ID
const deleteEmployee = (employeeId) => {
  const params = {
    TableName: tableName,
    Key: {
      id: employeeId,
    },
  };

  return dynamodb.delete(params).promise();
};

// Example usage
const main = async () => {
  try {
    // Create a new employee
    await createEmployee({
      id: '1',
      firstName: 'John',
      lastName: 'Doe',
    });

    // Get an employee
    const employee = await getEmployee('1');
    console.log('Get Employee:', employee);

    // Update an employee's information
    await updateEmployee({
      id: '1',
      firstName: 'Jane',
      lastName: 'Doe',
    });

    // Delete an employee
    await deleteEmployee('1');
    console.log('Employee deleted.');
  } catch (error) {
    console.error('Error:', error);
  }
};

main();
```

Please note that this code is a basic example and assumes you have the AWS credentials configured with the necessary permissions to perform operations on the DynamoDB table. Be sure to replace `'YOUR_ACCESS_KEY'`, `'YOUR_SECRET_ACCESS_KEY'`, and the example data with your own values.

Additionally, you should handle errors and implement appropriate error handling in a production application.
