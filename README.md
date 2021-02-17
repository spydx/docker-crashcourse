# Docker CrashCourse

Mini crash course for Docker and a simple C# application

Goal is to build a docker container and use docker-compose to utilize the docker container in the project.
## YouTube Resources

[Docker in 100 Seconds](https://www.youtube.com/watch?v=Gjnup-PuquQ&ab_channel=Fireship)

[Docker Tutorial for Beginners](https://www.youtube.com/watch?v=3c-iBn73dDE&ab_channel=TechWorldwithNana)

[Kubernetes Explained in 100 Seconds](https://www.youtube.com/watch?v=PziYflu8cB8&ab_channel=Fireship)


## The Weather API

We are making a simple Weather Api that is exposing this address.
[WeatherApi](http://localhost:5000/swagger/index.html)
Our container will expose this service on port 8080 instead.

This project is using C#, but we don't need any tools on our machine to build this.
We are going to use the docker to do this for us.
We are going to change some code to utilize a external DB.

## Important Docker commands

```sh
docker ps                              // show all running containers
docker build --tag <name>:<tag> .      // build according to the Dockerfile
docker run -p <hostpost>:<containerport> -name <nameyourcontainer> <name>:<tag>

docker images prune                 // remove unused images
docker container
docker container list 
docker container inspect            // log into running container
```

## Create the container for the API

Test that the container is working

### Dockerfile

### Build steps

### Running it localy

## Create a Docker-Compose file

### Write the first draft

### Changing the code

### Adding a service

### Verifying that everything works
