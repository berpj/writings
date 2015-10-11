I'm currently working a lot with [Docker](https://docker.com) for my new startup [Datacube.io](https://datacube.io), so I thought it would be a good idea to share some things I have learned on the subject.

### A Todo application
For this article, we are going to host a very simple Todo application. It lets you create tasks, which are composed of a _title_ and some _content_.

Here is what it looks like:
![](http://i.imgur.com/0tfEz0K.png)

You can clone [this git repository](https://github.com/berpj/docker-architecture-demo) to get the source code and all the other files used in this article.

The application is composed of a front-end in AngularJS, created around a _service_ called _API_ (see [api.js](https://github.com/berpj/docker-architecture-demo/blob/master/web/app/scripts/services/api.js)). The back-end is a regular REST API created with Ruby on Rails and its awesome scaffolding feature (`rails generate scaffold Todo title:string content:text`). The actual *todos* are stored in a Postgres database.

### Architecture
We want to create a 3-tier web architecture composed of a web server (AngularJS), an application server (Ruby on Rails) and a database server (Postgres).

Here is the architectural diagram:
![](http://i.imgur.com/6bSJGHQ.png)

### Creating the containers

Let's define the three containers using Dockerfiles.

> A Dockerfile is a script used by Docker to build a container, step-by-step, layer-by-layer, automatically from a source base image.

**AngularJS container**

This container is based on the [official Nginx image](https://registry.hub.docker.com/_/nginx/). It comes with Nginx installed with default settings. We just need to add a few *RUN* lines to install some components and build the site.

*./web/Dockerfile:*

    FROM nginx:latest

    # Download packages
    RUN apt-get update
    RUN apt-get install -y curl \
                           git  \
                           ruby \
                           ruby-dev \
                           build-essential

    # Copy angular files
    COPY . /usr/share/nginx

    # Installation
    RUN curl -sL https://deb.nodesource.com/setup | bash -
    RUN apt-get install -y nodejs \
                           rubygems
    RUN apt-get clean
    WORKDIR /usr/share/nginx
    RUN npm install npm -g
    RUN npm install -g bower
    RUN npm install -g grunt-cli
    RUN gem install sass
    RUN gem install compass
    RUN npm cache clean
    RUN npm install
    RUN bower --allow-root install -g

    # Building
    RUN grunt build

    # Open port and start nginx
    EXPOSE 80
    CMD ["/usr/sbin/nginx", "-g", "daemon off;"]

**Rails container**

The official [Ruby on Rails image](https://registry.hub.docker.com/_/rails/) is very useful in the sense that you only have to add a few lines to run your rake commands, and all the other things are taken care of by default.

*./app/Dockerfile:*

    FROM rails:onbuild

    # Create and migrate DB
    RUN bundle exec rake db:create
    RUN bundle exec rake db:migrate

    # Start rails server
    CMD ["bundle", "exec", "rails", "server", "-b", "0.0.0.0"]

**Postgres container**

[macadmins/postgres](https://registry.hub.docker.com/u/macadmins/postgres/) is a Postgres setup that accepts remote connections from Docker IPs (172.17.0.1/16). So we can directly use this image, there is no need to create a new Dockerfile.

### Composing the application

Instead of running every container by individual Docker commands, let's create a [Docker Compose](https://docs.docker.com/compose/) file to define how our application works.

> Docker Compose is a tool for defining and running complex applications with Docker.

*./docker-compose.yml:*

    web:
      image: berpj/web-demo
      ports:
        - "80:80"
    app:
      image: berpj/app-demo
      links:
        - db
      ports:
        - "3000:3000"
      environment:
        RAILS_ENV: production
        DB_HOST: db
        DB_PORT: 5432
        DB_ADAPTER: postgresql
        DB_DATABASE: todo
        DB_USER: todo
        DB_PASS: mypass
    db:
      image: macadmins/postgres
      environment:
        DB_NAME: todo
        DB_USER: todo
        DB_PASS: mypass

This file defines each container by name, pointing to the images used for the build (for the ease of use I built these images in Dockerhub from the Dockerfiles we saw earlier). In addition, it contains the container links, ports to open, and even environment variables for each of them.

### Deploying

Let's use [Docker Machine](https://github.com/docker/machine).

> Docker Machine makes it really easy to create Docker hosts on your computer and on cloud providers.

We are going to deploy this application on [RunAbove](https://runabove.com), a new IaaS provider built on OpenStack. Start by creating an account if you don't have one already. 

Go into your [security settings](https://cloud.runabove.com/horizon/project/access_and_security/) and allow TCP ports 3000 (for the Ruby on Rails API) and 2376 (for Docker).

To set your credentials in your environment, download your [Open RC file](https://manager.runabove.com/horizon/project/access_and_security/api_access/openrc/) and execute:
     
    $ source XXXXXXXX-openrc.sh # The file you just downloaded

Then you need to set the availability zone where you want to deploy your instance. You can choose between SBG-1 (Strasbourg, France) and BHS-1 (Beauharnois, Canada). In this example we choose SBG-1:

    $ export OS_REGION_NAME=SBG-1
  
Deploying a new instance with a Docker engine is done with this single command using Docker Machine:
       
        $ docker-machine create \
          -d openstack \
          --openstack-flavor-name="ra.intel.sb.m" \
          --openstack-image-name="Ubuntu 14.10" \
          --openstack-net-name="Ext-Net" \
          --openstack-ssh-user="admin" \
          prod

Here, we just created a SandBox M instance with an Ubuntu 14.04 image. All the available instance types and images are listed in your [OpenStack Horizon dashboard](https://cloud.runabove.com/horizon/).

Then we tell our Docker client to use this remote server:

    $ eval "$(docker-machine env prod)"

Finally, all we have to do to run our application is:

    $ docker-compose up -d

Now, we have a Nginx server serving static files, and a Rails server, talking to a Postgres database. Each running in their own container! If we hit our host URL on port 80 we have our Todo application, up and running.

### What next?

From here, the logical step would be to scale our architecture by creating a [Docker Swarm](https://github.com/docker/swarm) cluster. But for now, linked containers are always scheduled on the same host by Docker Compose, so it would not work (see [github.com/docker/[...]/SWARM.md](https://github.com/docker/compose/blob/master/SWARM.md)).

Over the next few months, Docker's network abstraction is going to be improved and using Docker Compose and Docker Swarm together should be very powerful.
To follow on our example, we could add a [HAProxy](https://registry.hub.docker.com/_/haproxy/) in front, create a Docker Swarm cluster, and execute Docker Compose against it. It would let us do things like:

    $ docker-compose scale web=2 app=3

Docker is definitely revolutionising the cloud computing world.