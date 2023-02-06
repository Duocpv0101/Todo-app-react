# React Todo App

This is a sample react todo app done step-by-step.

## Instructions

First clone this repository.
```bash
$ git clone https://github.com/kabirbaidhya/react-todo-app.git
```
Install dependencies. Make sure you already have [`nodejs`](https://nodejs.org/en/) & [`npm`](https://www.npmjs.com/) installed in your system.
```bash
$ npm install # or yarn
```

Run it
```bash
$ npm start # or yarn start
```
Build it
```bash
$ npm build
```

## Setting up your AWS account:
### 1. Create an OpenID Connect Identity provider
The first step is to create an OpenID Connect (OIDC) identity provider in your AWS Account. This will allow Github to identify itself.

* Got to the IAM console -> Identity providers
* Click Add new provider
* Select OpenID Connect
* Provider Url: https://token.actions.githubusercontent.com (Don't forget to click Get Thumbprint)
* Audience: sts.amazonaws.com
* Add tags if you want to and click Add Provider

### 2. Create a role

You now need to create a role that Github will be able to assume in order to access the resources it needs to control.

* Go back to IAM and select Roles
* Create a **new Role**
* Chose **Web Identity**, select the Identity provider you created in the previous step, and its audience.
* Click Next:**Permissions**
* Go back to IAM Roles and select the created **Role**. Choose **Trust Relationships** and **Edit Trust Relationship**.
#### The final result will look like this:
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::1234567890:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
        },
        "StringLike": {
          "token.actions.githubusercontent.com:sub": "repo:[your-org]/[your-repo]:*"
        }
      }
    }
  ]
}
```
### 3. Configure Github action workflow
* Your Github workflow requires additional permissions in order to be able to use OIDC. Add the following at the top of your workflow's YML file. You can also add it at the job level to reduce the scope if needed.
```
permissions:
  id-token: write # required to use OIDC authentication
  contents: read # required to checkout the code from the repo
```
* Add this step to generate credentials before doing any call to AWS:
```
- name: configure aws credentials
  uses: aws-actions/configure-aws-credentials@v1
  with:
    role-to-assume: arn:aws:iam::1234567890:role/your-role-arn
    role-duration-seconds: 900 # the ttl of the session, in seconds.
    aws-region: us-east-1 # use your region here.
```
The configure AWS credentials step will use the OIDC integration to assume the given role, generate short-lived credentials, and make them available to the current job.