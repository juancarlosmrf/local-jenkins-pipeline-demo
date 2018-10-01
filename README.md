# This is what you need:

- Git client installed.
- Docker installed and working in your laptop (this is out of this instructions).
- A Jenkins running in docker locally (We will explain it below).
- The proper rights (ssh access key) for your local Jenkins docker user to pull from your *local* git repo (We will explain it below).
- A Jenkins pipeline project that pulls from your local git repository (We will explain it below).
- A "local-git" user in your local Jenkins (it does not need to be admin ;-D).
- A git project with a *post-commit* web hook that triggers the pipeline project using the "local-git" user (review the hooks_template dir).

# This is how you can do it:
## Download de code of the demo in your home directory.
```
cd $HOME
git clone git@github.com:juancarlosmrf/local-jenkins-pipeline-demo.git
```

## Jenkins Docker.
You can find a Dockerfile and a docker-compose.yml file in the root of the demo repository.

Run your local jenkins environment with docker-compose:
```
cd $HOME/local-jenkins-pipeline-demo
docker-compose ps
docker-compose down
docker-compose up -d
docker-compose ps
```

Everything you do in your local jenkins will be stored in the folder /var/tmp/jenkins_home and preserved between restarts.

## Create a ssh access key in your docker jenkins

This is a very important part for this to work (code repos checkout).
First we start the docker container (we did it with docker-compose) and create a bash shell to it:

```
cd $HOME/local-jenkins-pipeline-demo
docker-compose exec local-jenkins /bin/bash
```

You are now into the docker container (local Jenkins), you can see this by something like *"jenkins@b82713f4fad2:/$"* in your terminal (the hash after the @ will be different in your case).

### Create the ssh key:
```
jenkins@b82713f4fad2:/$ ssh-keygen
```
Press enter/enter/enter... on all questions until you get the prompt back.

Copy the key to your computer/laptop:
Enable the SSH remote access in your laptop ( sudo systemsetup -setremotelogin on ).


**Note:** Change "myuser" for your local user in your laptop.
```
jenkins@b82713f4fad2:/$ ssh-copy-id myuser@host.docker.internal
```
**Remember:**
myuser = your username and host.docker.internal is the host address to your computer from within the docker container.

You will have to type your password at this point.!!!!!

Now lets try to complete the loop by ssh-ing to your computer from within the docker container.
```
jenkins@b82713f4fad2:/$ ssh myuser@host.docker.internal
```
This time you should **not** need to enter you password. If you do so... something went wrong and you have to try again :-(

You will now be in your laptop's home folder. Try ls and take a look.

Do not stay here since we have a chain of ssh shells that we need to get out of.!!!
```
$ exit
jenkins@b82713f4fad2:/$ exit
```
Right! Now we are back and ready to continue.

## Install your Local Jenkins

You will find your local Jenkins in your browser at http://localhost:8787.

First time you point your browser to your local Jenkins your will see the Installation Wizard.
Defaults are fine, be sure you install the pipeline plugin during the setup though.

**Note:** to get the "initialAdminPassword":
```
docker-compose logs | grep -A5 -B5 initialAdminPassword
```

#### Setup your jenkins:

You are an admin user by default. You can verify your rights in this URL:
http://localhost:8787/configureSecurity

Please verify that the checkbox Prevent Cross Site Request Forgery exploits is unchecked. (Since this Jenkins is only reachable from your computer this isn't such a big deal)

#### Add the local-git user:

We need to allow the git hook to login to the local Jenkins. Just to see and build jobs is sufficient (but for this demo, this user can be an admin too). Therefore we create a user called "local-git" with password "Password123".

Point your browser to http://localhost:8787/securityRealm/addUser and add "local-git" as username and "Password123" as password. Click on [ Create User ].

#### Create the pipeline project
We assume we have our git enabled project with the Jenkinsfile in it is called "local-jenkins-pipeline-demo" and is located at **/home/myuser/local-jenkins-pipeline-demo**

In your Local Jenkins (http://localhost:8787) add a new **pipeline project**.
I named it "pipelinedemo" for reference.

Click on New Item in the Jenkins menu:
- Name the project "pipelinedemo"
- Click on Pipeline
- Click [ OK ]
- Tick the checkbox Poll SCM in the Build Triggers section. Leave the Schedule empty.
In the Pipeline section:
- Select Pipeline script from SCM
In the Repository URL field:
- enter myuser@host.docker.internal:local-jenkins-pipeline-demo/.git (**Note:** be aware with the username !!!)
- in the Script Path field enter "Jenkinsfile" (we assume it is in the root of the repos)
Save the "pipelinedemo" project
Build the "pipelinedemo" manually once (this is needed for the Poll SCM to start working).

## Create the git hook (in local git repository)
Got to the local git cloned code directory and copy the hook in hooks_template directory to the **"/home/myuser/local-jenkins-pipeline-demo/.git/hooks"** folder and create a file called post-commit, and do it executable.
```
cd $HOME/local-jenkins-pipeline-demo
cp hooks_template/post-commit .git/hooks/
chmod +x .git/hooks/post-commit
ls -la .git/hooks/post-commit
```

#### Test the post-commit hook:

````
$HOME/local-jenkins-pipeline-demo/.git/hooks/post-commit
```
Check in Jenkins if your "pipelinedemo" project was triggered.

Finally make some arbitrary change to your project, add the changes and do a commit.
This will now trigger the pipeline in your local Jenkins.


* http://localhost:8787/configureSecurity/ CSRF Protection to Off

Happy Demo!
