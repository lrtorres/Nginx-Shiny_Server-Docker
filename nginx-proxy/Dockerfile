#name of container: stats-docker-shiny
FROM ubuntu:16.04

# Necessary for add-apt-repository
RUN apt-get update
RUN DEBIAN_FRONTEND=noninteractive apt-get install -y python-software-properties
RUN DEBIAN_FRONTEND=noninteractive apt-get install -y software-properties-common

RUN apt-get -y --quiet --no-install-recommends install apt-transport-https

#RUN add-apt-repository -y ppa:nginx/stable
RUN add-apt-repository -y ppa:nginx/development
RUN apt-get update 

RUN apt-get -y --quiet --allow-unauthenticated --no-install-recommends install \
					nginx=1.11.9-0+xenial0 \
                    && apt-get clean \
                    && rm -rf /tmp/* /var/tmp/*  \
                    && rm -rf /var/lib/apt/lists/*

RUN chown -R www-data:www-data /var/lib/nginx

# This will allow nginx to "write" its logs to stdout.  Needed to use the host syslog.
RUN ln -sf /dev/stdout /var/log/nginx/access.log \
    && ln -sf /dev/stderr /var/log/nginx/error.log

ADD ./etc/nginx/nginx.conf /etc/nginx/nginx.conf

#volume for Shiny Apps and static assets. Here is the folder for index.html(link) and sample apps.
VOLUME ["/etc/nginx/certs", "/etc/nginx/conf.d", "/var/log/nginx", "/var/www/html"]

WORKDIR /etc/nginx

# to allow access from outside of the container  to the container service
# at that ports need to allow access from firewall if need to access it outside of the server. 
EXPOSE 80 443

CMD ["nginx"]
