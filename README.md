# .NET Worker Docker Sample

This sample demonstrates how to build container images for .NET console apps. You can use this samples for Linux and Windows containers, for x64, ARM32 and ARM64 architectures.

The sample builds an application in a [.NET SDK container](https://hub.docker.com/_/microsoft-dotnet-sdk/) and then copies the build result into a new image (the one you are building) based on the smaller [.NET Docker Runtime image](https://hub.docker.com/_/microsoft-dotnet-runtime/). You can test the built image locally or deploy it to a [container registry](../push-image-to-acr.md).

The instructions assume that you have cloned this repo, have [Docker](https://www.docker.com/products/docker) installed, and have a command prompt open within the `docker-circleboom-dotnet-worker` directory within the repo.

## Build a .NET image

You can build and run a .NET-based container image using the following instructions:

```console
docker build --pull -t docker-circleboom-dotnet-worker .
```

You can use the `docker images` command to see a listing of your image, as you can see in the following example.

```console
% docker images
REPOSITORY                        TAG       IMAGE ID       CREATED          SIZE
docker-circleboom-dotnet-worker   latest    58f444224541   27 minutes ago   187MB
docker101tutorial                 latest    552e99028728   3 months ago     27.5MB
```

To immediately run your image:
```
docker run --rm docker-circleboom-dotnet-worker
```

> Since this app is based on [`dotnet worker` template](https://devblogs.microsoft.com/aspnet/net-core-workers-as-windows-services/), it will immediately print the output on the console when you run it.
```
info: docker_circleboom_dotnet_worker.Worker[0]
      Worker running at: 01/26/2021 20:12:43 +00:00
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Production
info: Microsoft.Hosting.Lifetime[0]
      Content root path: /app
info: docker_circleboom_dotnet_worker.Worker[0]
      Worker running at: 01/26/2021 20:12:44 +00:00
info: docker_circleboom_dotnet_worker.Worker[0]
      Worker running at: 01/26/2021 20:12:45 +00:00
info: docker_circleboom_dotnet_worker.Worker[0]
      Worker running at: 01/26/2021 20:12:46 +00:00
```

To create a `docker container` with this newly created image;
```
% docker container create --name cb-dotnet-worker docker-circleboom-dotnet-worker
```

Start your container
```
% docker start cb-dotnet-worker
```

See the logs
```
docker logs cb-dotnet-worker -f
```

## The logic to build the image is described in the [Dockerfile](Dockerfile), which follows.

```Dockerfile
# https://hub.docker.com/_/microsoft-dotnet
FROM mcr.microsoft.com/dotnet/sdk:5.0 AS build
WORKDIR /source

# copy csproj and restore as distinct layers
COPY *.csproj .
RUN dotnet restore

# copy and publish app and libraries
COPY . .
RUN dotnet publish -c release -o /app --no-restore

# final stage/image
FROM mcr.microsoft.com/dotnet/runtime:5.0
WORKDIR /app
COPY --from=build /app .
ENTRYPOINT ["dotnet", "docker-circleboom-dotnet-worker.dll"]
```

The `sdk:5.0` and `runtime:5.0` tags are both multi-arch tags that will result in an image that is compatible for the given chip and OS. These simple tags (only contain a version number) are great to get started with Docker because they adapt to your environment. We recommend using an OS-specific tag for the runtime for production applications to ensure that you always get the OS you expect. This level of specification isn't needed for the SDK in most cases.

This Dockerfile copies and restores the project file as the first step so that the results of those commands can be cached for subsequent builds since project file edits are less common than source code edits. Editing a `.cs` file, for example, does not invalidate the layer created by copying and restoring project file, which makes subsequent docker builds much faster.

> Note: See [Establishing docker environment](../establishing-docker-environment.md) for more information on correctly configuring Dockerfiles and `docker build` commands.

## Build an image for Alpine, Debian or Ubuntu

Please refer to [Dotnet Docker repo to learn more about how to build for other platforms](https://github.com/dotnet/dotnet-docker/blob/master/samples/dotnetapp/README.md).

## More Samples

* [.NET Docker Samples](../README.md)
* [.NET Framework Docker Samples](https://github.com/microsoft/dotnet-framework-docker/blob/master/samples/README.md)


## Some Docker commands

```
// Create docker containers
# docker container create --name cb-dotnet-worker docker-circleboom-dotnet-worker

// List active containers
# docker ps

// Start a container
# docker start cb-dotnet-worker

// See the logs
# docker logs cb-dotnet-worker -f

// Restart an individual container
# docker restart cb-dotnet-worker

// Stop an individual container
# docker stop cb-dotnet-worker

// Remove an individual container
docker rm cb-dotnet-worker

// Remove a docker image
docker rmi docker-circleboom-dotnet-worker 
```
