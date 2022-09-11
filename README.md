# EC2gitContinuousDeployment
steps to deploy git master commits to AWS EC2 instance automatically

- Create AWS ec2 instance of type Amazon Linux 2 AMI
- Connect to the instance
- Install Docker:
  - `sudo yum update -y`
  - `sudo amazon-linux-extras install docker`
  - `sudo usermod -a -G docker ec2-user`
  -  reboot instance and reconnect
  - `sudo service docker start`
  - `docker --version`
- Install docker-compose:
  - `sudo curl -L "https://github.com/docker/compose/releases/download/1.26.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`
  - `sudo chmod +x /usr/local/bin/docker-compose`
  - `docker-compose --version`
- Install go:
  - `wget https://storage.googleapis.com/golang/getgo/installer_linux`
  - `chmod +x ./installer_linux`
  - `./installer_linux`
  - `source /home/ec2-user/.bash_profile`
  - `go version`
- Install git:
  - `sudo yum install git -y`
- Install webhook:
  - `go get github.com/adnanh/webhook`
  - `go build github.com/adnanh/webhook`
  - `ls` webhooks folder should appear.
- Connect to githup with ssh and clone project repo:
  - `ssh-keygen -t rsa -b 4096 -C "your_email@example.com"`
  - `eval "$(ssh-agent -s)"`
  - `ssh-add ~/.ssh/id_rsa`
  - `cat .ssh/id_rsa.pub` to print content in terminal
  -  copy conent and add new ssh key at github
  - `git clone <repo>`
- Create github webhook:
  -  open repo in github > settings > webhooks > add new webhook
  -  payload url `http://yourserver:9000/hooks/redeploy-webhook` can change redeploy-webhook to any other id
  -  generate and paste any secret
  -  continue
- Create hooks.json file in ec2 server:
  - `nano hooks.json`
  -  paste content after editing below
  -  change project folder name ~~nest-aws-hook~~ to your project
  -  change id and secret to the ones created in the repo github hook 
```
[
  {
    "id": "redeploy-webhook",
    "execute-command": "/home/ec2-user/nest-aws-hook/rebuild.sh",
    "command-working-directory": "/home/ec2-user/nest-aws-hook",
    "trigger-rule": {
      "match": {
        "type": "payload-hash-sha1",
        "secret": "CHANGEME",
        "parameter": {
          "source": "header",
          "name": "X-Hub-Signature"
        }
      }
    }
  }
]
```
- `./webhook -hooks hooks.json -verbose` to run webhooks in terminal for testing
- `nohup ./webhook -hooks hooks.json -verbose -hotreload &` to run in background after closing the terminal

> Now every push to the master branch (condition could change as need be) will trigger auto rebuild at the server
> Health of the current webhook could be checked at repo > settings > webhooks
> Good Luck with CD!!!!!!!!
