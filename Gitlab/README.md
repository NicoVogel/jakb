# Gitlab CI/CD with docker

This is a simple setup guide to get Gitlab-CE, Gitlab-Runner, Apache Achiva and Maven up and running for java. 

It is also the first time, that I did someting with CI and therefore this is probably not the best solution, but it works.

## Requirements

- docker

## Setup Gitlab container 

The following command will create a gitlab container and runs it. **Replace the placeholder for hostname before running the command.**

> further information: [gitlab docker wiki](https://docs.gitlab.com/omnibus/docker/#run-the-image)

````shell
sudo docker run --detach \
  --hostname <gitlab.example.com> \
  --publish <public port>:443 --publish <public port>:80 --publish <public port>:22 \
  --name gitlab \
  --restart always \
  --volume /srv/gitlab/config:/etc/gitlab \
  --volume /srv/gitlab/logs:/var/log/gitlab \
  --volume /srv/gitlab/data:/var/opt/gitlab \
  gitlab/gitlab-ce:latest
````

## Setup gitlab-runner

The following steps are required to get the gitlab-runner working. I personally preffer the alpine images, because they are small, but you can also use the tag *latest* instead of *alpine*.

**1. Create the Gitlab-Runner container**

The gitlab-runner needs access to the docker itselfe to start container, therfore the docker.sock is maped.

````shell
docker run -d --name gitlab-runner --restart always \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:alpine
````

> [further information about the gitlab runner image](https://docs.gitlab.com/runner/install/docker.html#docker-image-installation-and-configuration)

**2. Get Conatiner IP from Gitlab**

This step is required for the following step. It returns the gitlab container IP

````shell
sudo docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' gitlab
````

**3. Update Gitlab external URL**

This step is required so that the gitlab-runner is able to pull the code from gitlab. Connect to the gitlab container: 

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
external_url '<something inside here>'
````

Now insert the gitlab container IP from step 3. To apply the changes save the file and execute the following command

````shell
gitlab-ctl reconfigure
````
> [more information about gitlab configuration](https://docs.gitlab.com/omnibus/settings/configuration.html#configuring-the-external-url-for-gitlab)

> you can close the console with **exit**

**4. Get Runner Token from Gitlab**

The registration token for a gitlab-runner can be found on the gitlab page. There are the following three types of runner:

- Shared runner (*is globally availiable*)
    - location: **http://\<localhost>:<80>/admin/runners** 
    - under the topic **Setup a shared Runner manually**
- Group runner (*is for all projects within a group availiable*)
    - location: **http://\<localhost>:<80>/groups/\<group name>/-/settings/ci_cd**
    - expand **Runner**
    - under the topic **Setup a shared Runner manually**
- Project runner (*is only for one project*)
    - location: **http://\<localhost>:<80>/\<group name>/\<project name>/settings/ci_cd** 
    <br>or<br> **http://\<localhost>:<80>/\<user name>/\<project name>/settings/ci_cd**
    - expand **Runner**
    - under the topic **Setup a shared Runner manually**

There is the URL which will be used in the following setup, as well as the registration token.

**5. Register runner images**

These images will be used to run the stages which are defined in the *.gitlab-ci.yml*.

First connect to the gitlab-runner:

````shell
sudo docker exec -it gitlab-runner sh
````

Now replace the placeholder with the information from step 2 and 5. The tags are in this case not realy needed, because of the option *--run-untagged*.

````shell
gitlab-runner register \
 --non-interactive \
 --executor "docker" \
 --docker-image alpine:latest \
 --url "<gitlab container ip>" \
 --registration-token "<gitlab runner token>" \
 --description "<runner name>" \
 --tag-list "<tags comma seperated>" \
 --run-untagged \
 --locked="false"
````

> [more about runner registration](https://docs.gitlab.com/runner/register/index.html#one-line-registration-command)

> you can close the console with **exit**

**6. [optional] Runner configuration**

Edit the config with the following command **(ROOT IS REQUIRED, otherwise the file will be empty)**

````shell
sudo nano /srv/gitlab-runner/config/config.toml
````

Information:
> [toml datatype](https://github.com/toml-lang/toml)

> [advanced config doku](https://docs.gitlab.com/runner/configuration/advanced-configuration.html)

**7. Check if the runner is connected correctly**

To do so go to the same locations where you copied the registration token from setp 4 and check if there is a runner which wasent there before. The runner will also contain the description and tags which you defied while registration.

If the symbol infront of the runner name is a green dot, then your runner is ready. Otherwise (for example a warning triangle) then you have to check what is wrong with the runner. Helpfull are the gitlab-runner logs. The following command will output the logs:

````shell
sudo docker logs [-f] gitlab-runner
```` 

> **-f** is used to continiusly see the logs. To exit this state, use [STRG] + [C]. Without -f you get the latest logs

## Setup project 

### Create project and open it in eclipse

- open gitlab and click on create project, fill the form and press "create project"
- copy the git repo link
- open eclipse and select the git view
- click "clone repository"
- enter your credentials and click "next", "finish"
- rightclick on the git repo and select "import projects"
- select master and import projects
- switch back to the java view

### Setup project structure

Create the following folder:
- .m2
- src/main/java
- src/test/java

Add the files which are located in the **files** folder into the following destination folder (filename -> directory):
- settings.xml -> /.m2
- mvn_build.launch -> /
- pom.xml -> /
- .gitignore -> /

Make the following changes to the files:
- pom.xml -> update groupId, artifactId, version, packaging, name and description
- settings.xml -> nothing is needed
- mvn_build.launch -> we change this later
- .gitonore -> the gitignore was generated from https://www.gitignore.io/api/java,maven,eclipse,intellij 

Now use the git bash or eclipse and commit and push the chages.

## Setup Apache Achiva

setup achiva

## last configuration steps

update values in mvn_build.launch and in gitlab CI values

## add .gitlab-ci.yml to the project

create .gitlab-ci.yml for your needs

DONE.

# Issues I came across

## job is not distributed to a runner which is availiable

My problem was that my job was in the pipline, but the gitlab-runner which was availiable did not want to communicate with gitlab.

**Solution**

My runner did not have the *--run-untagged* and my job did not have any tags, therefore my runner didn't receive the job.

## gitlab-runner: config.toml empty after runner registration

I opend the config.toml from the gitlab-runner with *nano config.toml* and the file was empty. So I startet writing the config by myselfe and broke it while doing so...

**Solution**

After deleting the container and running a new instance, I wrote **sudo** infront of nano out of habbit (because all docker commands require it) and the file was not empty. I noticed later that **sudo** did the job.  

## runner registration failed

Here I had two issues:

1. Runner didn't like the url I defined. **solution**: the url has to start with 'http://' or 'https://' and no port is allowed!

2. I chose a *--token* and the registration which works fine without a token didn't work anymore. **Solution** still don't know the solution, but it is not required. This way you chould choose a name for the runner instead of the generated ID.

# References I used to get it running

- The gitlab documentation
- The docker documentation
- [Gitlab ci docker tutorial](https://about.gitlab.com/2018/02/05/test-all-the-things-gitlab-ci-docker-examples/)
- [Gitlab with gitlab-runner with docker-compose](https://github.com/jeshan/gitlab-on-compose)
- [Youtube tutorial on Gitlab CI](https://www.youtube.com/watch?v=34u4wbeEYEo&list=PLaFCDlD-mVOlnL0f9rl3jyOHNdHU--vlJ&index=1)
- [How to deploy Maven projects to Artifactory with GitLab CI/CD](https://docs.gitlab.com/ee/ci/examples/artifactory_and_gitlab/)
- [How to integrate between Apache Archiva and Maven](https://www.mkyong.com/maven/how-to-integrate-between-apache-archiva-and-maven/)