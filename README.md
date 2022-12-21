# image-lamda
# LAB - 401-D49 Lab-17

## Project: Lamda + S3

### Author: Lewis Benson

### Problem Domain

Create an S3 bucket, Create a lamda function with a trigger that creates an images.json when a file is uploaded to the bucket. The JSON will include the image's meta data. 

### Links and Resources

[images.json](https://numsandstuff.s3.amazonaws.com/images.json)


### Setup

#### `.env` requirements (where applicable)

There are no env requirements

- PORT is set to process.env or 3001.
  Application will run locally or deployed to AWS without setting any parameters manually

#### How to initialize/run your application (where applicable)

- nodemon
  app.js is the primary JavaScript file for the server, and is located in the root folder.

#### Steps Create a new Bucket

Click services on the nav bar

Hover `storage` -> click on S3

Click the create bucket button on the right.

Leave the default settings, add a name, and unblock public access

Create Button

Click permissions tab, edit the bucket policy

Add a new policy

Click the button on the right to add actions. search for S3 then add GetObject

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
It should look like the above codeblock

The resource is found above the menu

#### Steps Create a a new Lambda


Click services on the nav bar

Hover `compute` -> click on Lambda

Click Create function

check default rules

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
Code used in the Lambda 
select S3 Rekognition tests
Click the tests, set the name and key inside the nested propertys to make the bucket name and a file you uploaded (if you did not upload one, do this now)


#### Steps Create a a new Trigger

inside the newly created Lambda click the New Trigger button

Select the S3

Add a suffix for the file type you want to be triggered. 

create the trigger


#### Steps Setup permission

Highlight Security, Identity, & Compliance -> Click IAM

click on roles. 

You should see the role you created when you made the new Lambda (see the name of the lamda function in the role name)

Check the policy listed, and click attach policys 

Add the customer managed roles with the long name, and lamda in the name

attach AmazonS3FullAccess by searching S3

Everything should be ready to accept file uploads, and perform tests at this point. 


#### Features / Routes

- Feature one: Created a bucket named numsandstuff

- Feature two: Created a Lamda function with a trigger to create a JSON. 

#### Tests

- How do you run tests?
On the lambda function console, click the test button with a test.jpg in the bucket

# cloud-server
