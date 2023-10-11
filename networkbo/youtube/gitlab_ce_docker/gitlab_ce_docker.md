## Step 1 Create a Linode Server




## Step 2 Install Docker

Update existing package list
`sudo apt update`

Install packages that let apt use HTTPS:
`sudo apt install apt-transport-https ca-certificates curl software-properties-common`

Add key for the official Docker repository:
`curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -`

Add Docker repository to APT sources:
`sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"`

Install Docker CE:
`sudo apt install docker-ce`

Confirm it's running:
`sudo systemctl status docker`

(Optional)
Add user to docker group:
`sudo usermod -aG docker ${USER}`

Apply group changes to user:
`su - ${USER}`

## Step 3 Manually launch GitLab

Make a directory to hold docekr run commands:
`mkdir sbin && cd sbin`

Make posgresql run command script and make it executable
```
$ sudo vi gitlab-postgresql

docker run --name gitlab-postgresql -d \
    --env 'DB_NAME=gitlabhq_production' \
    --env 'DB_USER=gitlab' --env 'DB_PASS=password' \
    --env 'DB_EXTENSION=pg_trgm,btree_gist' \
    --volume /srv/docker/gitlab/postgresql:/var/lib/postgresql \
    sameersbn/postgresql:12-20200524

$ sudo chmod +x gitlab-postgresql

$ ./gitlab-postgresql
```

Comfirm gitlab-postgresql is running
`docker ps`

Make redis run script
```
$ sudo vi gitlab-redis

docker run --name gitlab-redis -d \
    --volume /srv/docker/gitlab/redis:/data \
    redis:6.2

$ sudo chmod +x gitlab-redis

$ ./gitlab-redis
```

Comfirm gitlab-redis running
`docker ps`

Make gitlab container
```
$ sudo vi gitlab-start

docker run --name gitlab -d \
    --link gitlab-postgresql:postgresql --link gitlab-redis:redisio \
    --publish 10022:22 --publish 10080:80 \
    --env 'GITLAB_PORT=10080' --env 'GITLAB_SSH_PORT=10022' \
    --env 'GITLAB_SECRETS_DB_KEY_BASE=long-and-random-alpha-numeric-string' \
    --env 'GITLAB_SECRETS_SECRET_KEY_BASE=long-and-random-alpha-numeric-string' \
    --env 'GITLAB_SECRETS_OTP_KEY_BASE=long-and-random-alpha-numeric-string' \
    --volume /srv/docker/gitlab/gitlab:/home/git/data \
    sameersbn/gitlab:16.4.1

$ sudo chmod +x gitlab-start

$ ./gitlab-start
```

Comfirm gitlab is running
`docker ps`
