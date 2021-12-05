# Traefik Basic Configuration

### Requirements:

* Basic knowledge of how the Traefik reverse proxy operates. See [Traefik Docs](https://doc.traefik.io/traefik/)
* A server or VPS
* `docker` and `docker-compose` installed on server


## What does it do? 

This is a basic configuration for the Traefik reverse proxy. It's meant to remove some of the work of manually setting up Traefik from scratch by including best practices and using environment variables. 

## Why is it important? 

Starting out with Traefik can be difficult and confusing. This repository is a template you can use to get started more quickly. 

When beginner users first start off with Traefik, it isn't immediately clear whether you should use docker-compose labels (for the docker provider), a static configuration file, or a dynamic configuration file. The best solution is a combination of all three:

* Dynamic File Configuration: Use this to set up routers, middlewares, and other things that you will be making frequent changes to. These configurations update immediately upon saving the .yml file. *(Tip: Make multiple dynamic config files and label them by purpose! This way, if you briefly break the dynamic config for your development services, your production services don't go down as well.)* 


* Static File Configuration: Best used for setting up entrypoints, certificate resolvers, providers, which are all settings that will rarely change. If you break your dynamic configuration, everything entered into the static configuration will continue to run. 


* Docker Labels: These are most useful for creating services (so that Traefik can automatically detect the port assigned by the Docker daemon), but ideally should not be used for setting up routers and middlewares since they have poor readability and require restarting the docker service to take effect. 

## How do I use it? 

1. Clone this repository
2. Fill out the .env file to set your default certificate authority domain and email, and Cloudflare API keys
3. Edit the `docker-compose.yml` file for your application's stack, and for *every service* (every container) you want to expose, add these labels: 

```
labels: 
	- traefik.enable=true
	- traefik.http.services.<myservice>.loadbalancer.server.port=<portnumber>
```

For `<myservice>`, enter a *unique* service name. You will reference this name when setting up routers in the dynamic config file. 

For `<portnumber>`, enter the *container port* that must be opened. For instance, port 80 for a web application. 

To set a container name to a custom setting, use the `container_name:` key. 

4. Edit the `dynamic/backend.yml` file (or create your own .yml file in `dynamic/`) and create a router for each service you want to expose. Use the name of your container and add `@docker` to tell Traefik that the service you want to expose is a docker container.
5. Go to the `traefik` project directory and run `docker-compose up -d` to start the reverse proxy in background mode. Optional: Run `docker-compose logs -f` to follow the logs and check for configuration errors
6. Go to `example.com/traefik` to access the dashboard