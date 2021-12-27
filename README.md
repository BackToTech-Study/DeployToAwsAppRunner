# Deploy app to AWS AppRunner

## [Prerequisites](https://docs.aws.amazon.com/apprunner/latest/dg/setting-up.html)

1. AWS account
2. IAM user user with access key (**do not use the root user**)
3. [AWS CLI installed](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
5. [Docker installed](https://docs.docker.com/get-docker/)
6. Amazon ECR

## [Configure AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html)

1. Sign in to the AWS Management Console and open the [IAM console](https://console.aws.amazon.com/iam/)

2. In the navigation pane, choose Users.

3. Choose the name of the user whose access keys you want to create, and then choose the Security credentials tab. (If you are missing a user you can create one here)

4. In the Access keys section, choose Create access key.

5. To view the new access key pair, choose Show. You will not have access to the secret access key again after this dialog box closes. Your credentials will look something like this:
```
Access key ID: AKIAIOSFODNN7EXAMPLE
Secret access key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
```

6. Run `aws configure` and insert the required data
```
AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: us-west-2
Default output format [None]: json
```

## [Create and configure ECR](https://docs.aws.amazon.com/AmazonECR/latest/userguide/getting-started-cli.html)

1. [Create the ECR instance](https://eu-west-1.console.aws.amazon.com/ecr/create-repository)

2. From the AWS CLI authenticate to ECR. You only need to run this once or after a long period passed and the credentials need an update.

```
aws ecr get-login-password --region region | docker login --username AWS --password-stdin aws_account_id.dkr.ecr.region.amazonaws.com
```

3. Build the docker image: 
```
docker build -t hello-world .
```

4. Before going forward you could try and run the docker image localy to check for any failures

5. Tag the image:
```
docker tag hello-world:latest aws_account_id.dkr.ecr.region.amazonaws.com/hello-world:latest
```

6. Push the image:
```
docker push aws_account_id.dkr.ecr.region.amazonaws.com/hello-world:latest
```

## App runner

1. Create the AppRunner service

* Repository type: Container registry
* Provider: Amazon ECR (select the image uploaded at the previous step)
* Deployment trigger: Manual (Automatic costs extra)
* Create a new service role
