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

3. If you are missing a user you can create one here. See [Create a new User](#create-a-new-user)

4. If you already have a user you can add it to a group by selecting the user then in the `Groups` section you can choose `Add user to groups`.
If you are missing the desired group you can create one frm the `User groups` option on the right side menu.

5. To view the new access key pair, choose Show. You will not have access to the secret access key again after this dialog box closes. Your credentials will look something like this:
```
  Access key ID: AKIAIOSFODNN7EXAMPLE
  Secret access key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
```
* Or you can download the keys in CSV format
![DownloadCrdentials](https://user-images.githubusercontent.com/10427459/154866231-1958f933-2bb6-41ff-aba2-36233a6290da.png)


6. To update the local AWS CLI run `aws configure` and insert the required data
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

3. Create a published release build of your application than build a docker image using that application release build: 
```
  dotnet publish --configuration Release
  docker build -t hello-world .
```

In this example the DockerFile simply uses the content of the publish folder:
```
  FROM mcr.microsoft.com/dotnet/aspnet:4.0
  WORKDIR /app
  EXPOSE 9090

  ENV ASPNETCORE_URLS="http://+:9090"
  ENV ASPNETCORE_ENVIRONMENT: Production

  COPY bin/Release/net4.0/publish . 
  ENTRYPOINT ["dotnet", "hello-world.dll"]
```


4. Tag the image:
```
  docker tag hello-world:latest aws_account_id.dkr.ecr.region.amazonaws.com/hello-world:latest
```

5. Push the image:
```
  docker push aws_account_id.dkr.ecr.region.amazonaws.com/hello-world:latest
```

## App runner

1. Open the AppRunner console

![AppRunner](https://user-images.githubusercontent.com/10427459/154867722-633478be-d726-4d74-bb76-dfb34a9fcfd1.png)

2. Create a new service

* Repository type: Container registry
* Provider: Amazon ECR (select the image uploaded at the previous step)
* Deployment trigger: Manual (Automatic costs extra)
* Create a new service role
![AppRunnerOptions](https://user-images.githubusercontent.com/10427459/154867825-a7a9974c-f4ea-419c-ab4f-bb41e768d82d.png)

3. In the `Services` list select your service, then click the deploy button to deploy the previously loaded container.
It takes a couple of minutes to build but you get updated in the `Event log`
```
  02-21-2022 12:01:44 AM [AppRunner] Service status is set to RUNNING.
  02-21-2022 12:01:43 AM [AppRunner] Service update completed successfully. New configuration is deployed.
  02-21-2022 12:01:34 AM [AppRunner] Successfully routed incoming traffic to application.
  02-21-2022 12:00:42 AM [AppRunner] Health check is successful. Routing traffic to application.
  02-21-2022 12:00:04 AM [AppRunner] Performing health check on port '9090'.
  02-20-2022 11:59:55 PM [AppRunner] Provisioning instances and deploying image.
  02-20-2022 11:59:44 PM [AppRunner] Service status is set to OPERATION_IN_PROGRESS.
  02-20-2022 11:59:44 PM [AppRunner] Service update started.
```
# Optional steps

## Create a new User

* Enter user name and select option `Access key - Programmatic access`. 
* If you want to give the user access to the console also asign `AWS Management Console Access`

![CreateNewUserStep1](https://user-images.githubusercontent.com/10427459/154858779-d9f3b049-5ba2-43eb-9de2-4ebd0bbd6444.png)

* If you don't have a `Access Group` you can create one here.
* Make sure to add the `AWSAppRunnerFullAccess`, `AmazonEC2ContainerRegistryFullAccess` and `CloudWatchLogsReadOnlyAccess` policy

![CreateGroupAndAsignRights](https://user-images.githubusercontent.com/10427459/154865833-7d00b68f-d44f-4cb5-8886-4b81a2ff0ba0.png)

## Build the image with docker-compose
* build the release `dotnet publish --configuration Release`
* build the docker image `docker-compose build`
