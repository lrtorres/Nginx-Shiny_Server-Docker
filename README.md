### Nginx Frontend, Shiny Server multiple container backend.

## Synopsis
The Shiny-lb project was developed as an effort to leverage physical servers, Docker, and Shiny Server to create a load balanced, scalable Shiny Server deployment that can support hundreds of concurrent users.  The current community version of Shiny Server only supports one single R worker process that has issues supporting over 150 concurrent users; to extend its usability beyond this limitation, we deploy Docker containers that run a Shiny Server instance each, this in turn runs its own R worker process.

## Requirements
To build the proxy and node containers, ensure that you have the following:
  * Docker engine version 1.12.x or later.
  * Docker Compose 1.11.x or later (or any that can support version '2' docker-compose.yml syntax).
  * Make 4.1+ (optional)

## Building the containers
We provided a Makefile for each of container "blueprint"; it uses docker compose to build and can be executed by simply running "make" in the respective directory (e.g. shiny-nodes).  To do a manual build using Docker Compose:
 1. Make the configuration changes per your organization (see Testing...).
 2. Run 'docker-compose build' as the root user.
 3. When complete run 'docker-compose up -d' as the root user.
By default, the proxy uses port 3838 while the nodes will use 40038-40041.  The Docker Compose configuration 
will launch four node instances, each on one of the node ports listed prior

To verify that the containers are running, execute 'docker ps' as root - the output will be similar to the block below.  This assumes that all of the containers are running in the same physical server:
```
CONTAINER ID        IMAGE                     COMMAND                  CREATED             STATUS              PORTS                                         NAMES
40a920d93e95        nginxproxy_shiny_lb       "nginx"                  27 minutes ago      Up 27 minutes       0.0.0.0:3838->80/tcp, 0.0.0.0:3839->443/tcp   shiny_lb
5e9eda83c63e        shinynodes_shiny_node_3   "/bin/sh -c '/usr/bin"   39 minutes ago      Up 39 minutes       0.0.0.0:40040->3838/tcp                       shiny_node3
1627a7278e22        shinynodes_shiny_node_4   "/bin/sh -c '/usr/bin"   39 minutes ago      Up 39 minutes       0.0.0.0:40041->3838/tcp                       shiny_node4
651e450c74fc        shinynodes_shiny_node_2   "/bin/sh -c '/usr/bin"   39 minutes ago      Up 39 minutes       0.0.0.0:40039->3838/tcp                       shiny_node2
df0b7e63914b        shinynodes_shiny_node_1   "/bin/sh -c '/usr/bin"   39 minutes ago      Up 39 minutes       0.0.0.0:40038->3838/tcp                       shiny_node1
```

## Testing the out-of-the-box implementation.
In order to test this implementation two changes are required:
  * Add the name of the test server to /path/to/shiny-lb/nginx-lb/etc/nginx/nginx.conf
    * e.g. server_name foo.server.com
  * Add the name of the server(s) running shiny nodes, for testing purposes this can be the same server
    * e.g. server foo.server.com:40038 fail_timeout=300 max_conns=150;
      * server foo2.server.com:40038... etc.
    * please note that this cannot be 'localhost'.
  * Build the proxy and node containers.
Once the containers have been built, the test URL becomes http://foo.server.com:3838/ where foo.server.com is the FQDN of your test server.

## How to include custom shiny applications.
Since we run shiny nodes inside docker containers, the best-practice method to include custom Shiny Apps (provided the required R packages have been defined in the Dockerfile blue print) is to bind-mount
The following set of steps illustrates the process:
  * If a particular R package or library is required and not defined in the shiny-nodes `Dockerfile` add it to `/<source directory>/shiny-lb/shiny-nodes/Dockerfile`.
    * e.g. `RUN Rscript -e "install.packages('package_name', repos='repository>')"`  Where `package_name` is the name R package and `repository` is the R repository where it is stored `(e.g RCRAN)`.
  * Place your applications in a directory on each host running nodes unless the nodes use a network file system in common (e.g. `foo.server.01:/src/shiny/apps`, `foo.server.02:/src/shiny/apps` where foo.server.x is the host or hosts running shiny-node containers).
  * Define a volume in the docker-compose.yml file for each shiny docker container node that will be deployed:
    * e.g. `- /path/to/shiny/apps:/srv/shiny-server/custom:ro`
    * Where `/path/to/shiny/apps` is an arbitrary location where the shiny apps are stored, `/srv/shiny-server/custom` is an area that's available to every container deployed.  A custom area can be created, if necessary, by updating the shiny-server.conf file provided in /<source directory>/shiny-lb/shiny-nodes/etc/shiny-server/shiny-server.conf as per Shiny config documentation.
  * Test a container on each host to ensure applications are accessible, in the respective case to to the configuration above, the test URL would be `(http://foo.server.01:40038/custom/)`.

## Manually building containers without Docker Compose (TBD).
