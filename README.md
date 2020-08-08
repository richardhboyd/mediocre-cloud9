# mediocre-cloud9

# CodeBuild

## [CodeBuild Local](https://github.com/aws/aws-codebuild-docker-images)
I want to test my `buildspec.yml` file wasting a bunch of time/money on actually running CodeBuild.


```bash
git clone https://github.com/aws/aws-codebuild-docker-images.git
cd aws-codebuild-docker-images
cd al2/x86_64/standard/3.0
# cd al2/aarch64/standard/2.0/ If you are Matthew S. Wilson
docker build --tag codebuild:001 . 
# wait a literal eternity
DOCKER_ID=$(docker images codebuild:001 --format "{{.ID}}")
cd /your/local/code/directory
path/to/aws-codebuild-docker-images/local-builds/codebuild_build.sh -i $DOCKER_ID -a $(pwd)/artifacts -m -p isengard_example
```

## CDK

### Building CDK Constructs with Cloud9

```bash
# Get the latest version of Node
VERSION=$(nvm ls-remote --lts | grep Latest | tail -1 | grep -o "\(v[0-9][0-9]*\.[0-9][0-9]*\.[0-9][0-9]*\)")
nvm install $VERSION
# Just to be safe, let's grab the latest typescript
npm uninstall -g typescript
npm install -g typescript

sudo ln -s /usr/bin/pip-3.6 /usr/bin/pip3 # needed because of one of the tests inside CDK needs pip3

# Apparently dotnet is needed now too
sh -c "$(curl -fsSL https://dot.net/v1/dotnet-install.sh)"
export PATH=$PATH:/home/ec2-user/.dotnet

git clone https://github.com/[your_github_alias]/aws-cdk.git
cd aws-cdk/
git remote -v
git remote add upstream https://github.com/aws/aws-cdk.git
# If you've diverged, I usually check out the last common commit
# git checkout [HASH]
git checkout -b cloud9
git merge upstream/master
```

# AWS SAM

## Update SAM

```bash
# sudo localedef -i en_US -f UTF-8 en_US.UTF-8 # Likely not needed
export HOMEBREW_NO_ENV_FILTERING=1
CI=1 /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
test -d ~/.linuxbrew && eval $(~/.linuxbrew/bin/brew shellenv)
test -d /home/linuxbrew/.linuxbrew && eval $(/home/linuxbrew/.linuxbrew/bin/brew shellenv)
test -r ~/.bash_profile && echo "eval \$($(brew --prefix)/bin/brew shellenv)" >>~/.bash_profile
echo "eval \$($(/home/linuxbrew/.linuxbrew/bin/brew --prefix)/bin/brew shellenv)" >>~/.profile
npm uninstall -g aws-sam-local
sudo pip uninstall aws-sam-cli -y
sudo rm -rf $(which sam)
brew tap aws/tap
brew install aws-sam-cli
ln -sf $(which sam) ~/.c9/bin/sam
ls -la ~/.c9/bin/sam
```

# Python

## Update Python Version

```bash
sudo -u root -s
VERSION=3.7.4
INSTALL_DIR=/usr/local
sudo yum update -y
sudo yum install gcc openssl-devel bzip2-devel libffi-devel -y
wget https://www.python.org/ftp/python/${VERSION}/Python-${VERSION}.tgz
tar xzf Python-${VERSION}.tgz
cd Python-${VERSION}
./configure --prefix=${INSTALL_DIR}
make -j $(nproc)
make install -j $(nproc)
cd ../
### DANGER ZONE, this will break parts of Cloud9
# Remove old symlinks
rm -rf /etc/alternatives/pip
rm -rf /etc/alternatives/python
# make new symlinks
ln -s ${INSTALL_DIR}/bin/pip${VERSION:0:3} /etc/alternatives/pip
ln -s ${INSTALL_DIR}/bin/python${VERSION:0:3} /etc/alternatives/python
### END DANGER ZONE
exit
#TODO add /home/ec2-user/.local/lib/python3.8/site-packages to 'special' PATH
```

# GitHub and SSH

## SSH Keys
From my laptop I run

```bash
HOME_DIR=~
aws secretsmanager create-secret --name dev/github/richardhboyd --secret-string file://${HOME_DIR}/.ssh/github --region us-west-2 --profile federate
```

Then I can run this from my Cloud9 Environment to fetch and set the SSH Keys.

```bash
HOME_DIR=/home/ec2-user/
git config --global user.name "Richard Boyd"
git config --global user.email rhboyd@amazon.com
# Add your private key
aws secretsmanager get-secret-value --secret-id dev/github/richardhboyd --query "SecretString" --output text --region us-west-2 > ${HOME_DIR}.ssh/github

#restrict the access to the keys
chmod 400 ${HOME_DIR}.ssh/github

echo "IdentityFile ${HOME_DIR}.ssh/github" > ${HOME_DIR}.ssh/config
chmod 400 ${HOME_DIR}.ssh/config
```

# Hugo

Running Hugo on Cloud9

```
wget https://github.com/gohugoio/hugo/releases/download/v0.62.1/hugo_0.62.1_Linux-64bit.tar.gz
tar -xvzf hugo_0.62.1_Linux-64bit.tar.gz
sudo mv hugo /usr/local/bin/

PREVIEW_URL="https://$C9_PID.vfs.cloud9.us-west-2.amazonaws.com/"
hugo serve --bind=0.0.0.0 -p 8080 -b $PREVIEW_URL --appendPort=false --disableFastRender
```

# Extra

## Links to examples of people using Cloud9
- [Field Notes: Optimize your Java application for AWS Lambda with Quarkus](https://aws.amazon.com/blogs/architecture/field-notes-optimize-your-java-application-for-aws-lambda-with-quarkus/) by [Sascha Moellering](https://twitter.com/sascha242) and [Steffen Grunwald](https://twitter.com/steffeng)