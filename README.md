# API- DYNAMO & LAMBDA
# LAB - 401-D49 Lab-18

## Project: Restful API

### Author: Lewis Benson

### Problem Domain

Create a fully RESTful application using the AWS Gateway, with DynamoDB and Dynamoose as an ORM.

### Links and Resources

[Invoke URL:] (https://w4e42i2xic.execute-api.us-east-1.amazonaws.com/production)


### Setup

#### `.env` requirements (where applicable)

There are no env requirements.


#### How to initialize/run your application (where applicable)

Upload a jpg to the numsandstuff bucket; the lambda will be triggered by the upload and update the images.json file. 

#### Steps Create a new Bucket

Click services on the nav bar.

Hover `storage` -> click on S3

Click the Create bucket button on the right.

Leave the default settings, add a name, and unblock public access.

Create Button

Click the permissions tab, edit the bucket policy.

Add a new policy

Click the button on the right to add actions. Search for S3, then add GetObject

```js
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Statement1",
            "Effect": "Allow",
            "Principal": "*",
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::numsandstuff/*"
        }
    ]
}
```
It should look like the above code block

The resource is found above the menu.

#### Steps Create a new Lambda


Click services on the nav bar.

Hover `compute` -> click on lambda

Click Create function

Check default rules

Select Node 16x

use x86_64

Create the function

```js
const AWS = require('aws-sdk');

exports.handler = async (event) => {

  const bucketName = event.Records[0].s3.bucket.name;
  const objectKey = event.Records[0].s3.object.key;


  const s3 = new AWS.S3();
  let imagesJson;
  try {
    const imagesJsonResponse = await s3.getObject({
      Bucket: bucketName,
      Key: 'images.json'
    }).promise();
    imagesJson = JSON.parse(imagesJsonResponse.Body.toString());
  } catch (e) {
    if (e.code === 'NoSuchKey') {

      imagesJson = [];
    } else {
      throw e;
    }
  }
let found = false;

  const imageMetadata = {
    name: objectKey,
    size: (await s3.headObject({
      Bucket: bucketName,
      Key: objectKey
    }).promise()).ContentLength,
    type: (await s3.headObject({
      Bucket: bucketName,
      Key: objectKey
    }).promise()).ContentType
  };


  
  for (let i = 0; i < imagesJson.length; i++) {
    if (imagesJson[i].name === objectKey) {

      imagesJson[i] = imageMetadata;
      found = true;
      break;
    }
  }
  if (found === false) {

    imagesJson.push(imageMetadata);
  }


  await s3.putObject({
    Bucket: bucketName,
    Key: 'images.json',
    Body: JSON.stringify(imagesJson)
  }).promise();
  console.log(imageMetadata);
  return imageMetadata;
};
```
Code used in the lambda 
select S3 Rekognition tests
Click the tests, set the name and key inside the nested properties to make the bucket name and a file you uploaded (if you did not upload one, do this now)


#### Steps Create a new Trigger

Inside the newly created lambda, click the New Trigger button

Select the S3

Add a suffix for the file type you want to be triggered. 

Create the trigger


#### Steps Setup permission

Highlight Security, Identity, & Compliance -> Click IAM

Click on roles. 

You should see the role you created when you made the new lambda (see the name of the lambda function in the role name)

Check the policy listed, and click attach policies 

Add the customer-managed roles with the long name and lambda in the name

attach AmazonS3FullAccess by searching S3

Everything should be ready to accept file uploads and perform tests at this point. 


#### Features / Routes

- Feature one: Created a bucket named numsandstuff

- Feature two: Created a Lamda function with a trigger to create a JSON. 

#### Collaboration: I worked with Elias, Steven, and Seth to get out the primary lambda function written. 

#### Tests

- How do you run tests?
On the lambda function console, click the test button with a test.jpg in the bucket.

# cloud-server
