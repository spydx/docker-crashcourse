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

This project is using C#, but we don't need any tools on our machine to build this, since
we are going to use the docker to do this for us.

## Important Docker commands

```sh
docker start <containername>
docker stop <containername>
docker ps                              // show all running containers
docker build --tag <name>:<tag> .      // build according to the Dockerfile
docker run -p <hostpost>:<containerport> -name <nameyourcontainer> <name>:<tag>

docker images prune                 // remove unused images
docker container list               // list containers
docker container inspect            // log into running container <containername>
```

### Using Docker-compose

```sh
docker-compose up -d
docker-compose down
docker-compose build
```

## Create the container for the API

Test that the container is working

### Dockerfile

A Dockerfile is a recipe for how to build and/or run your application in a container.

```Dockerfile
# Choose what image and tag to use and give it a name
FROM <image>:<tag> as <aliasbuild>
# Set a working dir inside the container
WORKDIR /app

# Copy the sourcefiles we need (everything in tht source folder)
COPY . .

# Let's use C# SDK to build the API as a Relase (no debug symbols)
RUN dotnet build WeatherApi.csproj -c Release

## this ends the building of the software
## now we are to specify running container
FROM <image>:<tag> as <aliasrun>

# For demo purposes we want to set an enviromental variable used bu the running app
ENV ASPNETCORE_ENVIRONMENT Development

# The built release is in a subfolder
ARG /app/bin/Release/net5.0/

# Now this image need the compiled release files
# And we are to use the alias from the build section
# and the ARG to only copy the files we need to run it
COPY --from=aliasbuild ${ROOT} .

# We are to expose port 8080 for this project
EXPOSE 8080

# Final point, here we start the application
ENTRYPOINT ["dotnet", "WeatherApi.dll"]
```

### Building steps

Build steps are done to reduce the size of the end image.
E.g. if the application is written in Rust, keeping the build and the running image the same.
The size of the container will be huge. Meaning, every time there is a change, you will have to download all this locally to get it down.
Rust build image may, depending on your build, be 1Gb+ if not more.

This is why we within the `Dockerfile` use these aliases `FROM <image>:<tag> as builder` and then use `FROM <image>:<tag> as runner`. They can be named anything, this is just to illustrate.

Note: If your application is not compiled, and don't need a build step, e.g. Python Flask webpage, React app, then it would suffice with a runner only image.

With that said, lets build the container.
In the root of the `WeatherApi` project, where the `Dockerfile` is placed we are to run these commands.

#### Example command

```sh
docker build <DockerfileDir> --tag <nameofcontainer>:<versiontag>
#DockerfileDir here is /WeatherApi/
#nameofcontainer we will name weather-api
#versiontag we will name test
```

```sh
docker build . --tag weather-api:test
```

Result of this command should look something like this:

```sh
Sending build context to Docker daemon  8.957MB
Step 1/10 : FROM mcr.microsoft.com/dotnet/sdk:latest AS builder
latest: Pulling from dotnet/sdk
45b42c59be33: Already exists 
752dcc4c3a04: Pull complete 
5ccb476d6b8b: Pull complete 
513626bd05cb: Pull complete 
081e89d42aee: Pull complete 
d5f79a00b6f6: Pull complete 
1715fec659d7: Pull complete 
11ed275a0edd: Pull complete 
Digest: sha256:eda85a60aeb00e7904347d321095d8115f171dacc5d84feba4644ca4905c0204
Status: Downloaded newer image for mcr.microsoft.com/dotnet/sdk:latest
 ---> dd6f8be8f903
Step 2/10 : WORKDIR /app
 ---> Running in 5ab250f04ec7
Removing intermediate container 5ab250f04ec7
 ---> 67997e326d83
Step 3/10 : COPY . .
 ---> 184f04dd63b9
Step 4/10 : RUN dotnet build WeatherApi.csproj -c Release
 ---> Running in e4b4be6579c2
Microsoft (R) Build Engine version 16.8.3+39993bd9d for .NET
Copyright (C) Microsoft Corporation. All rights reserved.

  Determining projects to restore...
  Restored /app/WeatherApi.csproj (in 4.41 sec).
  WeatherApi -> /app/bin/Release/net5.0/WeatherApi.dll

Build succeeded.
    0 Warning(s)
    0 Error(s)

Time Elapsed 00:00:07.16
Removing intermediate container e4b4be6579c2
 ---> 5a2fc415934e
Step 5/10 : FROM mcr.microsoft.com/dotnet/runtime:5.0-buster-slim AS runner
5.0-buster-slim: Pulling from dotnet/runtime
45b42c59be33: Already exists 
752dcc4c3a04: Already exists 
5ccb476d6b8b: Already exists 
513626bd05cb: Already exists 
Digest: sha256:baf1927183229b74592f5e2ff57a960a75ea60ecdcd8a2568f9ccaa9e0554d30
Status: Downloaded newer image for mcr.microsoft.com/dotnet/runtime:5.0-buster-slim
 ---> 343709c142cc
Step 6/10 : ENV ASPNETCORE_ENVIRONMENT Development
 ---> Running in 204e77eb6f67
Removing intermediate container 204e77eb6f67
 ---> 1b752c65c750
Step 7/10 : ARG /app/bin/Release/net5.0/
 ---> Running in 92453894bb77
Removing intermediate container 92453894bb77
 ---> aeca689ee115
Step 8/10 : COPY --from=builder ${ROOT} .
 ---> eba4693af3fe
Step 9/10 : EXPOSE 8080
 ---> Running in 7db1b9e5362d
Removing intermediate container 7db1b9e5362d
 ---> 888d0decc312
Step 10/10 : ENTRYPOINT ["dotnet", "WeatherApi.dll"]
 ---> Running in 8ca23362cf5c
Removing intermediate container 8ca23362cf5c
 ---> 12042a3b5853
Successfully built 12042a3b5853
Successfully tagged weather-api:latest
>
```

### Running it localy

Since it was reported successfully build we can use this container.

Lets just run it.
When we do this, we wanna just give it a name for convenience and expose the ports it is supposed to use.

```sh
$ docker run -p 8080:80 --name weatherapi weather-api:latest

info: Microsoft.Hosting.Lifetime[0]
      Now listening on: http://[::]:80
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Development
info: Microsoft.Hosting.Lifetime[0]
      Content root path: /app

```

Open your favourite browser and go to this URL
http://localhost:8080/swagger/

## Create a Docker-Compose file


### Write the first draft

### Changing the code

### Adding a service

### Verifying that everything works
