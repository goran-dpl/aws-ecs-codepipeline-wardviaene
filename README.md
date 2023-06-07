# AWS CodePipeline
Using AWS CodeCommit and AWS CodeBuild to build a NodeJS docker image with the application code bundled \
and AWS CodeDeploy to deploy on Amazon ECS Fargate.
## Initiate terraform and create resources
```bash
terraform init
terraform apply
```

## Clone the sample application
```bash
git clone https://github.com/goran-dpl/docker-demo.git
```

## Copy the configuraion files to the app directory
```bash
cp app/config/* docker-demo/
cp app/scripts/create-new-task-def.sh docker-demo/
```

## Create and upload SSH public key to your IAM user
```bash
cd ..
ssh-keygen -f mykey
```

## Get the current user and upload the ssh public key
```bash
USER_ARN=$(aws sts get-caller-identity | jq '.Arn')
USER_NAME=$(jq --raw-output 'match("(.*user?)\/(.*$)").captures[1].string' <<< $USER_ARN)
SSH_KEY_ID=$(aws iam upload-ssh-public-key --user-name $USER_NAME --ssh-public-key-body file://mykey.pub | jq --raw-output '.SSHPublicKey.SSHPublicKeyId')
```

#### List all public keys
```bash
aws iam list-ssh-public-keys --user-name $USER_NAME | jq --raw-output '.SSHPublicKeys[].SSHPublicKeyId'
```

## Set up the remote CodeCommit repo
```bash
git remote -v
git remote remove origin
git remote add origin ssh://$SSH_KEY_ID@git-codecommit.eu-west-2.amazonaws.com/v1/repos/demo
ssh-add mykey
```

## Push the app code and trigger the pipeline
```bash
cd docker-demo
git add .
git commit -am "Deploy code"
git push -u origin master
```