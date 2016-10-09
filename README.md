# hello-ecosystem
This builds the entire ecosystem of services for the "Hello" project


# About
This is an attempt to create a barebones project based on a "microservices" architecture.
We show off this architecture by creating an API that leverages multiple services in order to build its responses.
In particular, making a GET request to http://localhost:3000/greeting should present you with a response built from several services working together.


# Project Architecture
This project follows the model of having an "API Gateway", which allows for a single API to your project, that then coordinates with the multitude of services in order to build its responses.

### Services
We have many services in play:
- one to handle requests for English words, and has a method to generate an English greeting: 
    - https://github.com/shaunpersad/hello-service-english
- one to handle requests for Hawaiian words, and has a method to generate an Hawaiian greeting:
    - https://github.com/shaunpersad/hello-service-hawaiian
- one to handle requests for Spanish words, and has a method to generate an Spanish greeting: 
    - https://github.com/shaunpersad/hello-service-spanish
- one to handle requests for reversed words, and has a method to generate the reverse of a word: 
    - https://github.com/shaunpersad/hello-service-reverse
- An API Gateway: 
    - https://github.com/shaunpersad/hello-api-gateway

### Request-response Flow
When the user makes a request to the "/greeting" endpoint, the API Gateway makes a call to the English, Hawaiian, and Spanish
services, which in turn make calls to the "reversed words" service. Each language service then takes their own internally generated greeting,
along with the reverse of that greeting (courtesy of the "reversed words" service), and sends them both back to the API Gateway,
which then concatenates and uses the responses from all three language services to build it's own response.

The end result is something like: "hello (olleh), hola (aloh), aloha (ahola) from the upside-down world!"

### Inter-service Communication
Any number of methods (including http requests) can be used to communicate between service to service (including the API Gateway).
In this case, we are using a message queue called [RabbitMQ](https://www.rabbitmq.com/getstarted.html), which sends all requests to a queue, where a service listening to that queue may pull down
and work on that request.  This allows us to spin up any number of each service, where only one of each kind of service would be handling any given request.
This is how our microservice-based architecture allows us to scale.
 
We are also leveraging a library called [Seneca](http://senecajs.org/), to handling the inter-service routing to individual methods in each service.


# Running this example:
1. Download and install Docker for your OS.
2. In the project root, run `bash scripts/run.sh`
3. Navigate to http://localhost:3000/hi to test that the API is up (you should receive a "hi" response).
4. Navigate to http://localhost:3000/greeting to test the microservices in action (you should receive a greeting).
5. Profit!

# Scaling this example:
- `docker-compose scale rabbitmq=1 api-gateway=1 service-reverse=3 service-english=2 service-hawaiian=2 service-spanish=2` (or any other amount of integers)
- Note: in this current setup, only one instance of rabbitmq or the api-gateway can be run, but any number of the "worker" services can be run.
- To stop the services started by `docker-compose scale`, run `docker-compose stop`

# TODO
- Each service should have its own set of tests.
- Proper branch management (currently everything works with master branches)
- Continuous Integration

# General Microservice notes:

### Prereqs:
- Download [Docker](https://www.docker.com/products/overview)
- Set up a [Docker Hub account](https://hub.docker.com/)


### Step by Step of how to set up a microservice project:
1. Create a `{project-name}` directory locally, and use as your working directory for the following steps.
2. Create a `{project-name}-ecosystem` repository.
3. Create a `{project-name}-api-gateway` repository.
	- Set up an automated build in Docker Hub.*
4. Create n number of `{project-name}-service-{service-name}` repositories.
	- Set up automated builds for each in Docker Hub.*
5. If necessary, create a `{project-name}-common` repository to store common utilities that can be shared across all services.

### Setting up the ecosystem:
1. Create a `docker-compose.yml`
2. Define all the service containers and other dependencies, using their names from Docker Hub as the image.

### Setting up the API gateway and other services:
1. Create a `Dockerfile` and a `docker-compose.yml` file each repo's root.
    - This `docker-compose.yml` file can overwrite specific pieces of the ecosytem's `docker-compose.yml` file, such that your local changes to a particular service get reflected across the entire ecosystem locally.
	- If working on a node project, be sure nodemon gets installed--its very useful.
	- If working on a node project, you can initialize your project using a node image instead of relying on having node installed on the host (which defeats the purpose of having VMS/containers in the first place): `docker run --rm -v $(pwd)/src:/usr/src/app -w /usr/src/app node:4.3.2 npm init --yes`

### Working on a service:
1. You MUST have the `{project-name}-ecosystem` repo next to the `{project-name}-service-{service-name}` repo that you are working on, so that you can spin up the ecosystem with your modified service.
2. (Optional) Update the ecosystem by running `docker-compose pull` in the `{project-name}-ecosystem` directory.
3. To view how your changes affect the entire ecosystem, run `docker-compose` with the "-f" flags to reference the ecosystem `docker-compose.yml` + your modified service's `docker-compose.yml`, which will build your local service's container on top of the ecosystem's version of that service.

### Running the "current" live ecosystem:
1. In the `{project-name}-ecosystem` directory, run `docker-compose up`.

### Setting up an automated build in Docker Hub
- https://docs.docker.com/docker-hub/builds/
