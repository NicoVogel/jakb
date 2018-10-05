# Gitlab CI/CD with docker

This is a simple setup guide to get Gitlab-CE, Gitlab-Runner, Apache Achiva and Maven up and running for java. 

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
    - expand **Runner**
    - under the topic **Setup a shared Runner manually**

There is the URL which will be used in the following setup, as well as the registration token.

**5. Register runner images**

These images will be used to run the stages which are defined in the *.gitlab-ci.yml*.

First connect to the gitlab-runner:

````shell
sudo docker exec -it gitlab-runner sh
````

Now replace the placeholder with the information from step 2 and 5. The tags are in this case not realy needed, because of the option *--run-untagged*. for more information 

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

> advanced config doku: https://docs.gitlab.com/runner/configuration/advanced-configuration.html

> you can close the console with **exit**




## List of abbreviations

abbreviations|meaning
---|---
CI|Continues Integration
CD|Continues Development
Gitlab-CE| Gitlab Community Edition