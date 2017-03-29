# Overview

DockerCI helps you developing CodeIgniter PHP Framework projects

[![](https://images.microbadger.com/badges/image/sebobo/shel.dockerflow.svg)](https://microbadger.com/images/sebobo/shel.dockerflow "Get your own image badge on microbadger.com")

DockerCI creates the necessary Docker containers (webserver, database, php, mail)
to run your CodeIgniter Framework project. The package provides a wrapper script in `bin/dockerci`
which simplifies the handling of docker and does all the configuration necessary.

We created this package to make development on CodeIgniter Framework projects easier and
to create a simple reusable package which can easily be maintained and serves well for the standard project.

Development will continue further as the package is already reused in several projects.
Contributions and feedback are very welcome.

## Install docker

    https://docs.docker.com/installation/ (tested with docker v1.9 - v1.12)

## Install docker-compose

We use docker-compose to do all the automatic configuration:

    http://docs.docker.com/compose/install/ (tested with docker-compose v1.5 - v1.6)

The repository contains a Dockerfile which will automatically be built in the
[docker hub](https://registry.hub.docker.com/u/manmath/dockerci/) after each change
and used by docker-compose to build the necessary containers.

## On a Mac or Windows

It has been tested working with docker for Mac but not yet with docker for Windows.
Feel free to try it out and let us know if you cannot wait.

## Install dockerci into your distribution

Add `manmath/dockerci` as dev dependency in your composer, using the latest stable release is highly recommended.

*Example*:

```
composer require --dev manmath/dockerci 1.0.*
```

## Run dockerci

    bin/dockerci up -d

The command will echo the url with which you can access your project. Add the hostname then to your `/etc/hosts`
and set the ip to your docker host (default for linux is 0.0.0.0) or your boot2docker ip. You can also use any
subdomain with `*.hostname` and it will point to the same server. What you need to do is to add exact subdomain name
to your `/etc/hosts`.

The parameter `-d` will keep it running in the background until you run:

    bin/dockerci stop

The default database configuration for your `database.php` is:

    $db['default'] = array(
        'hostname' => 'db',
        'username' => 'root',
        'password' => 'root',
        'database' => 'dockerci'
    );

Also note that there is a second database `dockerci_test` available for your testing context. The testing context url
would be `test.hostname` and this hostname should be added to your `/etc/hosts` too.

## Check the status

    bin/dockerci ps

This will show the running containers. The `data` container can be inactive to do it's work.

# Tips & Tricks

## Using different CI_ENV

    CI_ENV=Production bin/dockerci up -d

DockerCI also setup a sub-context for testing depends on the current context you are running. In the above example,
it would be `Production/Testing`. Anyway, you can only use the parent context with the `bin/dockerci` command. So when
there is a need to execute command for the testing context, you need to first get into `app` container and then call the
command prefixed by the context variable.

    CI_ENV=Production bin/dockerci up -d
    bin/dockerci run app /bin/bash

## Using MailHog to test mailing

    smtp_host = 'mail'
    smtp_port = '1025'

And open `MyCIProject:8025` in your browser (use your own hostname) to see your mails.

Send emails from your CodeIgniter app and have fun.

## Running a shell in one of the service containers

    bin/dockerci run SERVICE /bin/bash

SERVICE can currently be `app`, `web`, `data` or `db`.

## Access project url when inside `app` container

As of current docker doesn't support bi-directional link, you cannot access web container from app container.
But in some case you will need this connection. For example in behat tests without selenium, you need the url of
your site in `Testing` context while running the tests has to be done inside the `app` container.

DockerCI adds additional script after starting all containers to fetch the IP address of web container and
append it to `/etc/hosts` inside app container as below:

```
WEB_CONTAINER_IP    project-url
WEB_CONTAINER_IP    test.project-url
```

## Access database inside container from docker host

While you can easily login to shell of the `db` container with `bin/dockerci run db /bin/bash`
and execute your mysql commands, there are some cases that you want to run mysql commands directly
from your host without having to login to the `db` container first. One of the best use cases,
for example, is to access the databases inside the container from MySQL Workbench tool.
To be able to do that, we have mapped database port inside the container (which is `3306`) to your
host machine through `3307` port.

![Screenshot of MySQL Workbench interface](/Docs/MySQL-Workbench.png "MySQL Workbench interface")

## Attach to a running service

Run `bin/dockerci ps` and copy the container's name that you want to attach to.

Run `docker exec -it <containername> /bin/bash` with the name you just copied.
With this you can work in a running container instead of creating a new one.

## Check open ports in a container

    bin/dockerci run SERVICE netstat --listen

# Further reading

* [blog post on php-fpm](http://mattiasgeniar.be/2014/04/09/a-better-way-to-run-php-fpm/)
* [nginx+php-fpm+mysql tutorial](http://www.lonelycoder.be/nginx-php-fpm-mysql-phpmyadmin-on-ubuntu-12-04/)
* [Docker documentation](http://docs.docker.com/reference/builder/)
* [docker-compose documentation](http://docs.docker.com/compose)
* [nginx.conf for CI](https://www.nginx.com/resources/wiki/start/topics/recipes/codeigniter/)
* [boot2docker version which supports nfs](https://vagrantcloud.com/yungsang/boxes/boot2docker)
* [MailHog](https://github.com/mailhog/MailHog/)
