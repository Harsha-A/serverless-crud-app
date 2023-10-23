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
Certainly, if you prefer to use error-first callbacks (commonly used in Node.js for asynchronous operations), you can modify the code to utilize callbacks. Here's an example of the same CRUD operations on the DynamoDB table using error-first callbacks:

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
const createEmployee = (employee, callback) => {
  const params = {
    TableName: tableName,
    Item: employee,
  };

  dynamodb.put(params, (err, data) => {
    if (err) {
      callback(err);
    } else {
      callback(null, data);
    }
  });
};

// Function to get an employee by their ID
const getEmployee = (employeeId, callback) => {
  const params = {
    TableName: tableName,
    Key: {
      id: employeeId,
    },
  };

  dynamodb.get(params, (err, data) => {
    if (err) {
      callback(err);
    } else {
      callback(null, data.Item);
    }
  });
};

// Function to update an employee's information
const updateEmployee = (employee, callback) => {
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

  dynamodb.update(params, (err, data) => {
    if (err) {
      callback(err);
    } else {
      callback(null, data.Attributes);
    }
  });
};

// Function to delete an employee by their ID
const deleteEmployee = (employeeId, callback) => {
  const params = {
    TableName: tableName,
    Key: {
      id: employeeId,
    },
  };

  dynamodb.delete(params, (err, data) => {
    if (err) {
      callback(err);
    } else {
      callback(null, 'Employee deleted.');
    }
  });
};

// Example usage
const main = () => {
  // Create a new employee
  createEmployee({ id: '1', firstName: 'John', lastName: 'Doe' }, (err, data) => {
    if (err) {
      console.error('Error:', err);
    } else {
      console.log('Employee created:', data);
      
      // Get an employee
      getEmployee('1', (err, employee) => {
        if (err) {
          console.error('Error:', err);
        } else {
          console.log('Get Employee:', employee);
          
          // Update an employee's information
          updateEmployee({ id: '1', firstName: 'Jane', lastName: 'Doe' }, (err, updatedEmployee) => {
            if (err) {
              console.error('Error:', err);
            } else {
              console.log('Updated Employee:', updatedEmployee);

              // Delete an employee
              deleteEmployee('1', (err, message) => {
                if (err) {
                  console.error('Error:', err);
                } else {
                  console.log(message);
                }
              });
            }
          });
        }
      });
    }
  });
};

main();
```

This code structure uses error-first callbacks for each DynamoDB operation, providing more traditional asynchronous handling in Node.js. Replace `'YOUR_ACCESS_KEY'`, `'YOUR_SECRET_ACCESS_KEY'`, and the example data with your own values.






=================



Certainly! To add DynamoDB table creation and a sample getItem function to the Serverless Framework template, you can define an AWS Lambda function and the corresponding DynamoDB resources. Here's an extended `serverless.yml` file:

```yaml
service: employee-database

provider:
  name: aws
  runtime: nodejs14.x
  region: us-east-1 # Set your desired AWS region

resources:
  Resources:
    EmployeeTable:
      Type: AWS::DynamoDB::Table
      Properties:
        TableName: Employee
        AttributeDefinitions:
          - AttributeName: EmployeeID
            AttributeType: N
          - AttributeName: LastName
            AttributeType: S
          - AttributeName: Salary
            AttributeType: N
          - AttributeName: Email
            AttributeType: S
          - AttributeName: Address
            AttributeType: S
        KeySchema:
          - AttributeName: EmployeeID
            KeyType: HASH
          - AttributeName: LastName
            KeyType: RANGE
        ProvisionedThroughput:
          ReadCapacityUnits: 5
          WriteCapacityUnits: 5
        LocalSecondaryIndexes:
          - IndexName: SalaryIndex
            KeySchema:
              - AttributeName: EmployeeID
                KeyType: HASH
              - AttributeName: Salary
                KeyType: RANGE
            Projection:
              ProjectionType: ALL
        GlobalSecondaryIndexes:
          - IndexName: EmailIndex
            KeySchema:
              - AttributeName: Email
                KeyType: HASH
            Projection:
              ProjectionType: ALL
            ProvisionedThroughput:
              ReadCapacityUnits: 5
              WriteCapacityUnits: 5
          - IndexName: AddressIndex
            KeySchema:
              - AttributeName: Address
                KeyType: HASH
            Projection:
              ProjectionType: ALL
            ProvisionedThroughput:
              ReadCapacityUnits: 5
              WriteCapacityUnits: 5

  Functions:
    getItem:
      handler: getItem.handler
      events:
        - http:
            path: getItem
            method: GET

plugins:
  - serverless-offline
```

Now, let's define the JavaScript code for the `getItem` Lambda function. Create a directory for your function code (e.g., `getItem`) and add a `handler.js` file inside it:

```javascript
// getItem/handler.js

const AWS = require('aws-sdk');
const docClient = new AWS.DynamoDB.DocumentClient();

exports.handler = async (event) => {
  try {
    const params = {
      TableName: 'Employee',
      Key: {
        EmployeeID: parseInt(event.queryStringParameters.EmployeeID),
        LastName: event.queryStringParameters.LastName,
      },
    };

    const data = await docClient.get(params).promise();

    return {
      statusCode: 200,
      body: JSON.stringify(data.Item),
    };
  } catch (error) {
    console.error('Error:', error);
    return {
      statusCode: 500,
      body: JSON.stringify({ error: 'Internal Server Error' }),
    };
  }
};
```

This code defines a Lambda function named `getItem` that retrieves an employee's data based on `EmployeeID` and `LastName`. It expects these parameters as query string parameters in the HTTP GET request.

Finally, deploy your service using `serverless deploy`. You'll have a DynamoDB table and an API endpoint (`/getItem`) that allows you to fetch employee records.

Remember to adapt the code and configuration based on your specific requirements and add proper error handling and security measures for a production environment.


====

To create a dynamic DynamoDB table name and add an S3 bucket name to your Serverless Framework configuration, you can make use of Serverless Framework's variable and reference capabilities. Here's an updated `serverless.yml` file with those changes:

```yaml
service: my-serverless-app
frameworkVersion: '2'
useDotenv: true

provider:
  name: aws
  runtime: nodejs14.x
  stage: ${opt:stage, 'dev'}
  region: us-east-1
  vpc:
    securityGroupIds:
      - sg-xxxxxxxxxxxxxx  # Replace with your security group ID
    subnetIds:
      - subnet-xxxxxxxxxx  # Replace with your private subnet ID
  iamRoleStatements:
    - Effect: "Allow"
      Action:
        - "dynamodb:*"
      Resource:
        - "arn:aws:dynamodb:us-east-1:123456789012:table/${self:custom.dynamoTableName}"

functions:
  myFunction:
    handler: handler.myFunction
    events:
      - http:
          path: /myendpoint
          method: POST
          private: true  # This restricts access to VPC
    vpc:
      securityGroupIds:
        - sg-xxxxxxxxxxxxxx  # Replace with your security group ID
      subnetIds:
        - subnet-xxxxxxxxxx  # Replace with your private subnet ID
    environment:
      TABLE_NAME: ${self:custom.dynamoTableName}

custom:
  stage: ${self:provider.stage}
  dynamoTableName: ${self:provider.stage}-MyTable
  s3BucketName: ${self:provider.stage}-my-s3-bucket

resources:
  Resources:
    MyTable:
      Type: AWS::DynamoDB::Table
      Properties:
        TableName: ${self:custom.dynamoTableName}
        AttributeDefinitions:
          - AttributeName: id
            AttributeType: N
        KeySchema:
          - AttributeName: id
            KeyType: HASH
        ProvisionedThroughput:
          ReadCapacityUnits: 5
          WriteCapacityUnits: 5

    MySNSTopic:
      Type: AWS::SNS::Topic
      Properties:
        DisplayName: "My SNS Topic"
        TopicName: ${self:custom.dynamoTableName}-MySNSTopic

    MyS3Bucket:
      Type: AWS::S3::Bucket
      Properties:
        BucketName: ${self:custom.s3BucketName}

  Outputs:
    MySNSTopicArn:
      Value:
        Fn::GetAtt:
          - MySNSTopic
          - Arn
    MyS3BucketName:
      Value: ${self:custom.s3BucketName}
```

In this updated configuration:

1. The DynamoDB table name is dynamic and is based on the `${self:custom.dynamoTableName}` variable.

2. The S3 bucket name is also dynamic and is based on the `${self:custom.s3BucketName}` variable.

3. The `custom` section defines these variables with the format `${self:provider.stage}-MyTable` and `${self:provider.stage}-my-s3-bucket` to create unique resource names for each stage.

This configuration allows you to deploy the service to different stages with unique DynamoDB table and S3 bucket names.
