# cfn [![Build Status](https://travis-ci.org/Nordstrom/cfn.svg?branch=master)](https://travis-ci.org/Nordstrom/cfn) [![Coverage Status](https://coveralls.io/repos/github/Nordstrom/cfn/badge.svg?branch=master)](https://coveralls.io/github/Nordstrom/cfn?branch=master) [![JavaScript Style Guide](https://img.shields.io/badge/code_style-standard-brightgreen.svg)](https://standardjs.com) [![bitHound Overall Score](https://www.bithound.io/github/Nordstrom/cfn/badges/score.svg)](https://www.bithound.io/github/Nordstrom/cfn)

cfn makes the following AWS CloudFormation tasks simpler.
##### Create / Update Stack
* If the stack already exists, it Updates; otherwise, it Creates.
* Monitors stack progress, logging events.
* Returns a Promises.  Resolves when stack Create / Update is done, Rejects if there is an error.

##### Delete Stack
* Monitors stack progress, logging events.
* Returns a Promises.  Resolves when stack Create / Update is done, Rejects if there is an error.

##### Cleanup Stacks
* Use regex pattern to delete stacks.
* Include `daysOld` to delete stacks this old.

## Install
```
$ npm install cfn --save-dev
```

## Usage

### Create / Update
Use cfn to create or update an AWS CloudFormation stack.  It returns a promise.  You can use Node.js modules or standard
json or yaml for AWS CloudFormation templates.

```javascript

var cfn = require('cfn');


// Create or update (if it exists) the Foo-Bar stack with the template.js Node.js module.
cfn('Foo-Bar', __dirname + '/template.js')
    .then(function() {
        console.log('done');
    });

// Create or update the Foo-Bar stack with the template.json json template.
cfn('Foo-Bar', 'template.json');

// Create or update the Foo-Bar2 stack with the template.yml yaml template.
cfn('Foo-Bar2', 'template.yml');

```

### Delete
Delete a stack.

```javascript
// Delete the Foo-Bar stack
cfn.delete('Foo-Bar');
```

### Cleanup
Cleanup stacks based on regex and daysOld.

```javascript
// Delete stacks starting with TEST- that are 3 days old or more
cfn.cleanup({
    regex: /TEST-/,
    minutesOld: 60
})
    .then(function() {
        console.log('done')
    });
```

### Stack Exists
Returns a boolean if a stack exists or not
```javascript
//Returns boolean if stack name 'foo-bar' exists
cfn.stackExists('foo-bar')
    .then(function(exists){
        if (exists){
            //Do something
        }
    })
```

## API

### cfn(name|options[, template])
Creates or Updates a stack if it already exists.  Logs events and returns a Promise.

#### name
The name of the stack to Create / Update.  If the first arg is a string it is used as name.

#### options
Options object.  If the first arg is an object it will be used as options.

#### template
Path to template (js, yaml or json file), JSON object, serialized JSON string, or a YAML string. This is optional and if given will override options.template (if present).  This arg is helpful if the first arg is the name of the template rather than an options object.

##### options.name
Name of stack.

##### options.template
Path to template (json, yaml or js file), JSON object, serialized JSON string, or a YAML string. If the optional second argument is passed in it
will override this.

##### options.async
If set to true create/update and delete runs asynchronously. Defaults to false.

##### options.params
Interpolated parameters into JS module-wrapped CloudFormation templates (only should be used with js files).

A JS module-wrapped template example is shown below:
```javascript
module.exports = function (params) {
    return {
        AWSTemplateFormatVersion: '2010-09-09',
        Description: 'Test Stack',
        Resources: {
            testTable: {
                Type: 'AWS::DynamoDB::Table',
                Properties: {
                    ...
                    TableName: 'FOO-TABLE-' + params.env
                }
            }
        }
    };
};
```

Is deployed as follows:
```javascript
cfn({
    name: 'Test-Stack',
    template: 'template.js',
    params: { env: 'dev' }
});
```

##### options.cfParams
The standard AWS CloudFormation parameters to be passed into the template.

A standard AWS CloudFormation template in yaml format is shown below:
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Test Stack'

Parameters:
  env:
    Type: String
    Description: The environment for the application

Resources:
  testTable:
    Type: AWS::DynamoDB::Table
    Properties:
      ...
      TableName: !Sub FOO-TABLE-${env}
```

Is deployed as follows:
```javascript
cfn({
    name: 'Test-Stack',
    template: 'template.yml',
    cfParams: { env: 'dev' }
});
```

##### options.awsConfig
This allows you to pass any [config properties allowed by the AWS Node.js SDK](http://docs.aws.amazon.com/AWSJavaScriptSDK/guide/node-configuring.html)

```javascript
cfn({
    name: 'Foo-Bar',
    template: _dirname + '/template.js',
    awsConfig: {
        region: 'us-west-2'
        accessKeyId: 'akid',
        secretAccessKey: 'secret'
    }
}).then(function() {
    console.log('done');
});
```
