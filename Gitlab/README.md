# Gitlab CI/CD with docker

This is a simple setup guide to get Gitlab-CE, Gitlab-Runner, Apache Archiva and Maven up and running for java. 

It is also the first time, that I did something with CI and therefore this is probably not the best solution, but it works.

## Requirements

- Docker

## Topics


- [Setup](#setup)
    - [Setup Gitlab Conatiner](#setup-gitlab-container)
    - [Setup gitlab-runner](#setup-gitlab-runner)
        - [1. Create the Gitlab-Runner container](#1-create-the-gitlab-runner-container)
        - [2. Get Container IP from Gitlab](#2-get-container-ip-from-gitlab)
        - [3. Update Gitlab external URL](#3-update-gitlab-external-url)
        - [4. Get Runner Token from Gitlab](#4-get-runner-token-from-gitlab)
        - [5. Register runner images](#5-register-runner-images)
        - [6. \[optional\] Runner configuration](#6-optional-runner-configuration)
        - [7. Check if the runner is connected correctly](#7-check-if-the-runner-is-connected-correctly)
    - [Setup project](#setup-project)
    - [Setup Apache Archiva](#setup-apache-archiva)
    - [last configuration steps](#last-configuration-steps)
    - [add .gitlab-ci.yml to the project](#add-gitlab-ciyml-to-the-project)
    - [autocleanup of used gitlab-runner container](#autocleanup-of-used-gitlab-runner-container)
- [Issues I came across](#issues-i-came-across)
- [References I used to get it running](#references-i-used-to-get-it-running)

# Setup

## Setup Gitlab container

The following command will create and run a gitlab container. **Replace the placeholder for hostname and port before running the command.**

> further information: [gitlab docker wiki](https://docs.gitlab.com/omnibus/docker/#run-the-image)

````shell
sudo docker run \
  --detach \
  --hostname <gitlab.example.com> \
  --publish <public port>:443 \
  --publish <public port>:80 \
  --publish <public port>:22 \
  --name gitlab \
  --restart always \
  --volume /srv/gitlab/config:/etc/gitlab \
  --volume /srv/gitlab/logs:/var/log/gitlab \
  --volume /srv/gitlab/data:/var/opt/gitlab \
  gitlab/gitlab-ce:latest
````

## Setup gitlab-runner

The following steps are required to get the gitlab-runner working. I personally prefer the alpine images, because they are small, but you can also use the tag *latest* instead of *alpine*.


### 1. Create the Gitlab-Runner container

The gitlab-runner needs access to the docker itself to start the container, therefore the docker.sock is mapped.

````shell
sudo docker run \
  --detach \
  --name gitlab-runner \
  --restart always \
  --volume /srv/gitlab-runner:/etc/gitlab-runner \
  --volume /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:alpine
````

> [further information about the gitlab runner image](https://docs.gitlab.com/runner/install/docker.html#docker-image-installation-and-configuration)

### 2. Get Container IP from Gitlab

This step is required for the following [step 3](#3-update-gitlab-external-url). It returns the gitlab container IP address.

````shell
sudo docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' gitlab
````

### 3. Update Gitlab external URL

This step is required for the gitlab-runner in order to pull the code from gitlab. Connect to the gitlab container: 

````shell
sudo docker exec -it gitlab sh
````

Edit the gitlab options:

````shell
nano /etc/gitlab/gitlab.rb
````

Search for the following section (should be the first option)

````rb
## GitLab URL
##! URL on which GitLab will be reachable.
##! For more details on configuring external_url see:
##! https://docs.gitlab.com/omnibus/settings/configuration.html#configuring-the-external-url-for-gitlab
# external_url 'EXTERNAL GENERATED ID'
````

Now **uncomment the line** and insert the gitlab container IP address from [step 2](#2-get-container-ip-from-gitlab) as follows:

````rb
external_url 'http://<gitlab ip address>'
````

To apply the changes, save the file and execute the following command:

````shell
gitlab-ctl reconfigure
````
> [more information about gitlab configuration](https://docs.gitlab.com/omnibus/settings/configuration.html#configuring-the-external-url-for-gitlab)

> you can close the console with **exit**

### 4. Get Runner Token from Gitlab

The registration token for a gitlab-runner can be found on the gitlab page. There are the following three types of runner, **choose one**:

> if you are on the same machine, then you can use the docker gitlab ip with the port 80

- Shared runner (*is globally available*)
    - location: **http://\<localhost>:<80>/admin/runners** 
    - under the topic **Setup a shared Runner manually**
- Group runner (*is for all projects within a group available*)
    - location: **http://\<localhost>:<80>/groups/\<group name>/-/settings/ci_cd**
    - expand **Runner**
    - under the topic **Setup a shared Runner manually**
- Project runner (*is only for one project*)
    - location: **http://\<localhost>:<80>/\<group name>/\<project name>/settings/ci_cd** 
    <br>or<br> **http://\<localhost>:<80>/\<user name>/\<project name>/settings/ci_cd**
    - expand **Runner**
    - under the topic **Setup a shared Runner manually**

There you can find the URL which will be used in the following setup, as well as the registration token.

### 5. Register runner images

These images will be used to run the stages which are defined in the *.gitlab-ci.yml*.

First, connect to the gitlab-runner:

````shell
sudo docker exec -it gitlab-runner sh
````

Now replace the placeholder with the information from [step 2](#2-get-container-ip-from-gitlab) and [4](#4-get-runner-token-from-gitlab). The tags in this case are not really needed, because of the option *--run-untagged*.

````shell
gitlab-runner register \
 --non-interactive \
 --executor "docker" \
 --docker-image alpine:latest \
 --url "http://<gitlab container ip>" \
 --registration-token "<gitlab runner token>" \
 --description "<runner name>" \
 --tag-list "<tags comma seperated>" \
 --run-untagged \
 --locked="false"
````

> [more about runner registration](https://docs.gitlab.com/runner/register/index.html#one-line-registration-command)

> you can close the console with **exit**

### 6. [optional] Runner configuration

Edit the config with the following command **(ROOT IS REQUIRED, otherwise the file will be empty)**

````shell
sudo nano /srv/gitlab-runner/config.toml
````

Information:
> [toml datatype](https://github.com/toml-lang/toml)

> [advanced config doku](https://docs.gitlab.com/runner/configuration/advanced-configuration.html)

### 7. Check if the runner is connected correctly

To do so, go to the same location where you copied the registration token from [step 4](#4-get-runner-token-from-gitlab) and check if there is a runner which wasn't there before. The runner will also contain the description and tags which you have defined while registration.

If the symbol in front of the runner name is a green dot, then your runner is ready. Otherwise (for example a warning triangle) you have to check what is wrong with the runner. Helpful are the gitlab-runner logs. 

The following command will output the logs:

````shell
sudo docker logs [-f] gitlab-runner
```` 

> **-f** is used to continiusly see the logs. To exit this state, use [STRG] + [C]. Without -f you get the latest logs

## Setup project

### Create project and open in eclipse

- open gitlab and click on create project, fill the form and press "create project"
- copy the git repo link
- open eclipse and select the git view
- click "clone repository"
- enter your credentials and click "next", "finish"
- rightclick on the git repo and select "import projects"
- select master and import projects
- switch back to the java view

### Setup project structure

Create the following folders (starting from project folder):
- .m2
- src/main/java
- src/test/java

Add the files which are located in the **files** [folder of the current git repo](./files) into the following destination folder (filename -> directory):
- settings.xml -> /.m2
- mvn_build.launch -> /
- pom.xml -> /
- .gitignore -> /

Make the following changes to the files:
- pom.xml -> update groupId, artifactId, version, packaging, name and description
- settings.xml -> nothing is needed
- mvn_build.launch -> we change this later
- .gitonore -> add a dot infront of the filename

> the gitignore was generated from https://www.gitignore.io/api/java,maven,eclipse,intellij and the line which excludes maven *.launch files was removed. 

Now use the git bash or eclipse and commit and push the chages.

## Setup Apache Archiva

Start an archiva docker container with the following command:

````shell
sudo docker run \
  --detach \
  --restart always \
  --name archiva \
  --hostname archiva \
  --publish <public port>:8080 \
  --volume /srv/archiva:/archiva-data \
  xetusoss/archiva
````

### Users

# TODO
change this part

Connect to Archiva and create a user with read access and a user with read/write access.
The read user is used in the local maven build and the other user will be used from gitlab to store the files in the repository.

You can also use only one User, this is up to you.

### Settings

One important step is to allow redepolyment. Log in as root and open the *Repositories* tab. Click on the *pen* for the **internal** repository. There is a checkbox **Block Redeployments** make sure that it is **not checked**!

## last configuration steps

update values in mvn_build.launch (which is located in the root of the project folder) and in gitlab CI values
Open the mvn_build.launch and replace the **###value###** in the **mapEntry** nodes.

- MAVEN_REPO_URL: Archiva url with port (http://\<ip>:\<port>)
- MAVEN_REPO_USER: Archiva read-only user name
- MAVEN_REPO_PASS: Archiva read-only user password

Open gitlab and navigate to your project, than go to *Settings*, *CI / CD* and click on **Expand** at **Variables**. Enter the following key value pairs:

- MAVEN_REPO_URL: Archiva url with port (http://\<ip>:\<port>)
- MAVEN_REPO_USER: Archiva user name
- MAVEN_REPO_PASS: Archiva user password

## add .gitlab-ci.yml to the project

Now that everything is up and running, we can add the file which enables Gitlab-CI. Copy the file **.gitlab-ci.yml** into your project. Now after every commit a build, test and a deployment is done.

> [more about gitlab ci config](https://docs.gitlab.com/ce/ci/yaml/)


## autocleanup of used gitlab-runner container

Sadly the containers are not removed after usage. They will slowly fill up your space. You can check out a handy solution I found here: https://gitlab.com/gitlab-org/gitlab-runner-docker-cleanup

# Issues I came across

## job is not distributed to a runner which is availiable

My problem was that my job was in the pipeline, but the gitlab-runner which was available did not want to communicate with gitlab.

**Solution**

My runner did not have the *--run-untagged* and my job did not have any tags, therefore my runner didn't receive the job.

## gitlab-runner: config.toml empty after runner registration

I opened the config.toml from the gitlab-runner with *nano config.toml* and the file was empty. So I started writing the config by myself and broke it while doing so...

**Solution**

After deleting the container and running a new instance, I wrote **sudo** infront of nano out of habit (because all docker commands require it) and the file was not empty. I noticed later that **sudo** did the job.  

## runner registration failed

Here I had two issues:

1. Runner didn't like the url I defined. **Solution**: the url has to start with 'http://' or 'https://' and no port is allowed!

2. I chose a *--token* at the point of runner registration which unfortunately did not work. **Solution**: I still don't know the solution, but it is not required. This way you chould choose a name for the runner instead of the generated ID.

# References I used to get it running

- The gitlab documentation
- The docker documentation
- [Gitlab ci docker tutorial](https://about.gitlab.com/2018/02/05/test-all-the-things-gitlab-ci-docker-examples/)
- [Gitlab with gitlab-runner with docker-compose](https://github.com/jeshan/gitlab-on-compose)
- [Youtube tutorial on Gitlab CI](https://www.youtube.com/watch?v=34u4wbeEYEo&list=PLaFCDlD-mVOlnL0f9rl3jyOHNdHU--vlJ&index=1)
- [How to deploy Maven projects to Artifactory with GitLab CI/CD](https://docs.gitlab.com/ee/ci/examples/artifactory_and_gitlab/)
- [How to integrate between Apache Archiva and Maven](https://www.mkyong.com/maven/how-to-integrate-between-apache-archiva-and-maven/)