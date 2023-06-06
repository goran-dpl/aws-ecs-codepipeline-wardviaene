# Initiate terraform and create resources
```bash
terraform init
terraform plan/apply
```

# Clone the sample application
```bash
git clone https://github.com/goran-dpl/docker-demo.git
```

# Copy the configuraion files to the app directory
```bash
cd docker-demo
cp ../app/config/* .
cp ../app/scripts/create-new-task-def.sh .
```

# Create and upload SSH public key to your IAM user
```bash
cd ..
ssh-keygen -f mykey
```

# Get the current user and upload the ssh public key
cat mykey.pub
USER_ARN=$(aws sts get-caller-identity | jq '.Arn') 
USER_NAME=$(jq --raw-output 'match("(.*user?)\/(.*$)").captures[1].string' <<< $USER_ARN)
SSH_KEY_ID=$(aws iam upload-ssh-public-key --user-name $USER_NAME --ssh-public-key-body file://mykey.pub | jq --raw-output '.SSHPublicKey.SSHPublicKeyId')

# Check by listing all public keys
aws iam list-ssh-public-keys --user-name $USER_NAME | jq --raw-output '.SSHPublicKeys[].SSHPublicKeyId'

# Push your app code to GitCommit - Go to CodeCommit and copy the clone SSH URL
git remote -v
git remote remove origin
git remote add origin ssh://$SSH_KEY_ID@git-codecommit.eu-west-2.amazonaws.com/v1/repos/demo
ssh-add mykey

cd docker-demo
git add .
git commit -am "Deploy code"
git push -u origin master
