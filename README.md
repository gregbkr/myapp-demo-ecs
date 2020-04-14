# MyApp: a container in ECS

## Overview
- ECR: container registry
- CI: Codebuild build container and update task in ECS
- ECS: run container
- API gateway: to publish the app
- Infra as code: cloud formation

## Init 

- Set AWS to deploy in `eu-west-1`: `nano ~/.aws/config`
- First create the infra (Codebuild project & IAM role & S3 for SAM):

```
cd cloudformation
nano main.yml <-- edit with your needs
aws cloudformation create-stack --stack-name myapp-demo-ecs-infra-init --template-body file://main.yml --capabilities CAPABILITY_NAMED_IAM
... update-stack ... <-- if already created
```

## Deploy API

### Deploy manually
- Follow code in buildspec.yml to deploy manually (or use local build, see in annexes).

### Deploy via CI
- Edit ci with your needs: `nano buildspec.yml`
- Push code to master or develop for auto CI/CD

### Check
- You will get the url from the output of `sam deploy...`


## Destroy all
- Destroy using the right region: `aws cloudformation delete-stack --stack-name <YOUR_STACK_NAME> --region eu-west-1`

Todo:
- [ ] Cloudformation init (erc, codebuild, ecs)
- [ ] Build container
- [ ] CI update container
- [ ] Plug with API gateway
- [ ] Dev + Prod


## Annexes

### Local Codebuild: 
- Download + build docker [here](https://github.com/aws/aws-codebuild-docker-images/tree/master/ubuntu/standard/3.0)
- Download codebuild_build.sh [here](https://github.com/aws/aws-codebuild-docker-images/blob/master/local_builds/codebuild_build.sh)
- Edit `.env` to set the version to deploy [develop|master]
- Run local build: `./codebuild_build.sh -i aws/codebuild/standard:3.0 -a /tmp/artifacts -s . -e .env.production -c`