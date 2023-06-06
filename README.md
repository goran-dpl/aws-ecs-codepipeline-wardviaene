### Initiate terraform and create resources
terraform init
terraform plan/apply

# Clone the sample application
git clone https://github.com/goran-dpl/docker-demo.git

# Copy the configuraion files to the app directory
cd docker-demo
cp ../app/config/* .
cp ../app/scripts/create-new-task-def.sh .

# Create and upload SSH public key to your IAM user
cd ..
ssh-keygen -f mykey

# Get the current user and upload the ssh public key
cat mykey.pub
USER_ARN=$(aws sts get-caller-identity | jq '.Arn') 
USER_NAME=$(jq --raw-output 'match("(.*user?)\/(.*$)").captures[1].string' <<< $USER_ARN)
aws iam upload-ssh-public-key --user-name $USER_NAME --ssh-public-key-body file://mykey.pub
# List all public keys
aws iam list-ssh-public-keys --user-name $USER_NAME | jq --raw-output '.SSHPublicKeys[].SSHPublicKeyId'

# Push your app code to GitCommit - Go to CodeCommit and copy the clone SSH URL
cd docker-demo
git remote add origin https://git-codecommit.eu-west-2.amazonaws.com/v1/repos/demo
ssh-add ../mykey
git status
git add .
git commit -am "Deploy code"
git remove origin
git remote add origin ssh://<ssh_key_id>@git-code-commit.eu-west-2.amazonaws.com/v1/repos/demo
git push -u origin master




### Set up the remote CodeCommit repo
git remote -v

git remote -v 
git remote remove origin
git status