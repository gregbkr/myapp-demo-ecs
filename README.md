# MyApp: HA container in ECS

## Overview
This setup will deploy a redundant helloworld container on ECS fargate, with automatic CI/CD from AWS.
More info: you can find an overview of that setup on my [blog](https://greg.satoshi.tech/ci-cd-on-aws-elastic-container-service-ecs/)

### Infra
![Infra](./.github/images/myapp-ecs-infra.png)

- Cloud: AWS
- [ECS](https://aws.amazon.com/ecs): container orchestrator (on 2 availability zones for redundancy)
- [ECR](https://aws.amazon.com/ecr): container registry to store hello image
- App: a simple hello world in nodejs (folder `hello`)
- Code source: github
- Deployment: [CloudFormation](https://aws.amazon.com/cloudformation) describe all component to be deployed. One command line will setup 
- CI/CD: [Codepipeline](https://aws.amazon.com/codepipeline) to buid and deploy the container in ECS
the infra and return an url to access the application.

### CI/CD flow diagram

![CI/CD](./.github/images/myapp-ecs-cicd.png)
A simple `git push` from a developer in Github will launch the whole CI/CD process. Docker image will build and ECS will update to run that new image without any downtime.

## Deploy

### Prerequisites
Please setup on your laptop:
- AWS cli and AWS account to deploy in `eu-west-1`
- Docker and Compose
- Github personal token with `admin:repo_hook, repo` rights from [here](https://github.com/settings/tokens)


### Test app on your laptop
Check the app locally:
```
cd hello
docker-compose up -d
curl localhost 8080
```

### Deploy to AWS
- Set a unique project prefix, and deploy the infra:
```
cd cloudformation
export CF_DEMO_ENVIRONMENT=myapp-demo-ecs   <-- please change to your prefix!
./deploy.sh ${CF_DEMO_ENVIRONMENT} [GH username] [GH repo] [GH branch] [GH token]

Example:
./deploy.sh ${CF_DEMO_ENVIRONMENT} gregbkr myapp-demo-ecs master 21414af27e9f3f2eccaf68554459b5a8e1d17c5b
```

- Wait for the script to complete. If it takes more than 20 minutes, check cloudformation waiting component in event. It may be ECS who is wainting for the container to be up. Please check that the build was successful and the image present in ECR.

### Check
- Export both values:
```
export APP_URL=$(aws cloudformation \
   describe-stacks \
   --query 'Stacks[0].Outputs[?OutputKey==`WebServiceUrl`].OutputValue' \
   --stack-name ${CF_DEMO_ENVIRONMENT})

export CI_URL=$(aws cloudformation \
   describe-stacks \
   --query 'Stacks[0].Outputs[?OutputKey==`PipelineUrl`].OutputValue' \
   --stack-name ${CF_DEMO_ENVIRONMENT}
```
- Check the app with: `curl $APP_URL`

### CI-CD
- Change the ouput of the helloworld here: `nano hello/server.js` 
- Push code in github
- Check the status of CI-CD in your browser: `echo $CI_URL` 
- Then check again your app: `curl $APP_URL`


### Destroy all
- Destroy using the right region: `./delete-stacks.sh ${CF_DEMO_ENVIRONMENT}`


<em>Thank you [laser team](https://github.com/laser/cloudformation-fargate-codepipeline-ecs-refarch) for the bootstrap code!</em>
