# Step 7: Setup Docker 

## Setup Docker on Local Environment

Docker provides a container to run your applications isolated from the other processes running on host machine.

Install Docker Desktop based on your Operating System. [Download Docker](https://docs.docker.com/desktop/)
After you have installed the docker, run the docker from your application menu. You can see an empty Docker Dashboard 
opens up.

![](resources/9_setup_docker_1_dashboard.png)


**Why run docker?**

Because it helps you to standardise your installations regardless of your operating system and focus only on the steps 
necessary for you to run your applications.

**What does that even mean?**

Remember how you had to set several environment variables to run Java, install jdk, maven etc and perform different steps
as a prerequisite to fulfill before you can start writing and deploying your applications. With Docker, you can just do it 
using a single line by using pre-existing Docker Image.

**What is a Docker Image?**

A Docker image contains everything needed to run an application - all dependencies, configurations, scripts, binaries. There
are prexisting images in [Docker Hub](https://hub.docker.com/) that you can use to define installations/configurations/dependencies
like maven or java. 

Below will help you to fetch an image which has all the necessary configurations for maven and then you can simply run 
maven commands without getting into the hassle of performing installations steps on different machines.

```
FROM maven:3.9-amazoncorretto-17 AS build
```

**Why do we need Docker in our Java Applicaiton?**

Since you will be deploying your application on a cloud provider, it is easier to deploy a docker container 
than performing multiple steps on the cloud instances/servers. In our case we will deploy our application on Render
which does not provide support for Java but provides support for Docker images.

Learn more about Docker [here](https://docs.docker.com/get-started/)

## Running Java Application with Docker

In your case you will use two pre-existing images maven and java to build your application using maven and run your
application using the java Docker image. Using these two you will create your own Application's Image that you can push 
to Docker hub and deploy on Render.

To get started in your project repo, create a file named `Dockerfile`. For simplicity and avoid complications, 
keep the name as is. Add the following content to your Dockerfile.

```dockerfile
# build stage
FROM maven:3.9-amazoncorretto-17 AS build

WORKDIR /app

COPY . .
RUN mvn clean package

# run stage
FROM openjdk:17-jdk-slim

WORKDIR /app

COPY --from=build /app/target  target

EXPOSE 8080
ENTRYPOINT ["java", "-jar", "target/jsp-demo-jar-with-dependencies.jar"]
```

This file performs following actions: 

1. Pulls a maven image from DockerHub that contains all your configuration required to build your applicaitons
2. Sets a working director named app
3. COPY everything from your current path (.) to the Docker working director (.)
4. Compiles and packages your code, similar to what you do on your local environment that creates a target folder.
5. Pulls a Java Image from DockerHub to provide java runtime to run your application
6. Copy the target folder from your build stage (step 4) to the work directory
7. Expose port 8080, when your application start, the embedded tomcat container sets port 8080.
8. Run the jar using java -jar command as previously seen.

To build this docker image on your local machine, the docker application must be running. Else you will face an error.
To build the docker image, run below command 

```
docker build -t jsp-demo .
```

The option -t defines a tag for your docker image, in this case named as jsp-demo. The . in the end of the command signifies
where the Dockerfile can be found. Thus you need to run this command from the folder where you have Dockerfile. 
Once the image is built you can see this in your dashboard as well. 

![](resources/9_setup_docker_2_image.png)

To run the docker image on your local machine run below command - 
```
docker run -p 8090:8080 jsp-demo
```
The above command deploys the docker container from the image named jsp-demo created in previous step and maps the port 
8090 to 8080. the port 8080 is the port of embedded tomcat that you have exposed from your code. The port 8090 is the port where the 
docker container is listening to handle the requests. 

Alternatively, you can run the docker container from the dashboard as well. Also you can set any port of your choice 
and not just 8090
![](resources/9_setup_docker_3_run.png)

Once the container is running then if you type http://localhost:8090/ on your web browser,
you should be able to view your application deployed.

![](resources/9_setup_docker_4_8090.png)


## Pushing Docker Image to DockerHub

Create an account on [Docker Hub](https://hub.docker.com/). Verify your email. Create a private repo in DockerHub

![](resources/9_setup_docker_8_repo.png)

```{attention}
To secure code from plagiarism, please ensure that you have selected your repo as private.
```

run below commands

```
docker login -u <username>
docker image ls
docker tag <imageid> <username>/<reponame>:<image tag>
docker push <username>/<reponame>:<image tag>
```

![](resources/9_setup_docker_7_push.png)

You can verify in DockerHub that your image has been successfully pushed.

![](resources/9_setup_docker_9_tag.png)


```{admonition} What's Next
Please proceed to [Step 8: Deploy to Render](10_render_deploy.md).
```

