# Choose what image and tag to use and give it a name
FROM mcr.microsoft.com/dotnet/sdk:latest AS builder
# Set a working dir inside the container
WORKDIR /app

# Copy the sourcefiles we need (everything in tht source folder)
COPY . .

# Let's use C# SDK to build the API as a Relase (no debug symbols)
RUN dotnet build WeatherApi.csproj -c Release

## this ends the building of the software
## now we are to specify running container
FROM mcr.microsoft.com/dotnet/aspnet:latest AS runner

# For demo purposes we want to set an enviromental variable used bu the running app
ENV ASPNETCORE_ENVIRONMENT Development

# The built release is in a subfolder
ARG ROOT=/app/bin/Release/net5.0/

# Set a working directory for the new running container
WORKDIR /app

# Now this image need the compiled release files
# And we are to use the alias from the build section
# and the ARG to only copy the files we need to run it
COPY --from=builder ${ROOT} .

# We are to expose port 8080 for this project
EXPOSE 8080

RUN dir /app

# Final point, here we start the application
ENTRYPOINT ["dotnet", "WeatherApi.dll"]