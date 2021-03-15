# Python Docker Sample Application
Source: https://docs.docker.com/language/python/

## Tools Needed for Tutorial
* [Python version 3.8 or later](https://www.python.org/downloads/) or follow [Opensource Python installation](https://opensource.com/article/19/5/python-3-default-mac)
* [Docker running locally](https://docs.docker.com/desktop/)
* [An IDE or a text editor to edit files - VSCode](https://code.visualstudio.com/Download)

---

# Tutorial Instructions
## Build Images
### Directory Setup
Create a simple Python application using the Flask framework. Create a directory named `python-docker` and follow the steps to create a simple web server:
```
$ cd /path/to/python-docker
$ pip3 install Flask
$ pip3 freeze > requirements.txt
$ touch app.py
```
Add the following code to `app.py` to handle simple web requests:
```
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, Docker!'
```

### Test the Application
Start the application and make sure it's running properly. Run the following command in your working directory:
```
python3 -m flask run
```
The `-m` flag stands for `module_name`. To test that the application is working properly, open a new browser and navigate to `http://localhost:5000`. Looking at the terminal where the server is running, you should see similar requests in the server logs:
```
127.0.0.1 - - [14/Mar/2021 15:33:12] "GET / HTTP/1.1" 200 -
```

### Create a Dockerfile for Python
Now that we know that the application works properly, we'll create a Dockerfile. Run `touch Dockerfile` in the working directory.

Add the following line to the Dockerfile. This line tells Docker what base image we want to use for our application.
```
FROM python:3.8-slim-buster
```
Docker images can be inherited from other images. Therefore, instead of creating our own base image, we'll use the official Python image that already has all the tools and packages that we need to run a Python application. To learn more about creating your own base images, see [Creating base images](https://docs.docker.com/develop/develop-images/baseimages/).

To make things easier when running the rest of our commands, let's create a working directory. This instructs Docker to use this path as the default location for all subsequent commands. By doing this, we don't have to type out full file paths but can use relative paths based on the working directory.
```
WORKDIR /app
```
Usually, the first thing you do once you've downloaded a project written in Python is to install `pip` packages. This ensures that your application has all its dependencies installed.

Before we can run `pip3 install`, we need to get our `requirements.txt` file into our image. We'll use the `COPY` command to do this. The `COPY` command takes two parameters. The first parameter tells Docker what file(s) you would like to copy into the image. The second parameter tells Docker where you want that file(s) to be copied to. We'll copy the `requirements.txt` file into our working directory `/app`.
```
COPY requirements.txt requirements.txt
```
Once we have our `requirements.txt` file inside the image, we can use the `RUN` command to execute the command `pip3 install`. This works exactly the same as if we were running `pip3 install` locally on our machine, but this time the modules are installed into the image.
```
RUN pip3 install -r requirements.txt
```
The `-r` flag stands for `requirement`. At this point, we have an images that is based on Python version 3.8 and we have installed our dependencies. The next step is to add our source code into the image. We'll use the `COPY` command just like we did with our `requirements.txt` file above.
```
COPY . .
```
This `COPY` command takes all the files located in the current directory and copies them into the image. Now, all we have to do is to tell Docker what command we want to run when our image is executed inside a container. We do this using the `CMD` command. Note that we need to make the application externally visible (i.e. from outside the container) by specifying `--host=0.0.0.0`.
```
CMD ["python3", "-m", "flask", "run", "--host=0.0.0.0"]
```
Here's the complete Dockerfile.
```
FROM python:3.8-slim-buster

WORKDIR /app

COPY requirements.txt requirements.txt
RUN pip3 install -r requirements.txt

COPY . .

CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0"]
```
### Directory Structure
Just to recap, we created a directory in our local machine called `python-docker` and created a simple Python application using the Flask framework. We also used the `requirements.txt` file to gather our requirements, and created a Dockerfile containing the commands to build an image. The Python application directory structure would now look like:
```
python-docker
|____ app.py
|____ requirements.txt
|____ Dockerfile
```
### Build an Image
Now that we've created our Dockerfile, let's build our image. To do this, we use the `docker build` command. The `docker build` command builds Docker images from a Dockerfile and a "context". A build's context is the set of files located in the specified PATH or URL. The Docker build process can access any of the files located in this context.

The build command optionally takes a `--tag` flag. The tag is used to set the name of the image and an optional tag in the format `name:tag`. We'll leave off the optional `tag` for now to help simplify things. If you don't pass a tag, Docker uses "latest" as its default tag. You can see this in the last line of the build output.

Build the Docker image:
```
docker build --tag python-docker .
```
You'll see an output similar to this:
```
Sending build context to Docker daemon  79.36kB
Step 1/6 : FROM python:3.8-slim-buster
3.8-slim-buster: Pulling from library/python
6f28985ad184: Pull complete
f58644152280: Pull complete
456500070a3e: Pull complete
9229690cc337: Pull complete
47da0924d2de: Pull complete
Digest: sha256:1389669225e7fa05a9bac20d64551b6b6d84ee3200330d8d8de74c6d2314fdc7
Status: Downloaded newer image for python:3.8-slim-buster
 ---> 8671a55eb48a
Step 2/6 : WORKDIR /app
 ---> Running in 1a052cfb4361
Removing intermediate container 1a052cfb4361
 ---> 39a8620d399b
Step 3/6 : COPY requirements.txt requirements.txt
 ---> 22e8ab00bb5a
Step 4/6 : RUN pip3 install -r requirements.txt
 ---> Running in 842b3832f45e
Collecting click==7.1.2
  Downloading click-7.1.2-py2.py3-none-any.whl (82 kB)
Collecting Flask==1.1.2
  Downloading Flask-1.1.2-py2.py3-none-any.whl (94 kB)
Collecting itsdangerous==1.1.0
  Downloading itsdangerous-1.1.0-py2.py3-none-any.whl (16 kB)
Collecting Jinja2==2.11.3
  Downloading Jinja2-2.11.3-py2.py3-none-any.whl (125 kB)
Collecting MarkupSafe==1.1.1
  Downloading MarkupSafe-1.1.1-cp38-cp38-manylinux2010_x86_64.whl (32 kB)
Collecting Werkzeug==1.0.1
  Downloading Werkzeug-1.0.1-py2.py3-none-any.whl (298 kB)
Installing collected packages: MarkupSafe, Werkzeug, Jinja2, itsdangerous, click, Flask
Successfully installed Flask-1.1.2 Jinja2-2.11.3 MarkupSafe-1.1.1 Werkzeug-1.0.1 click-7.1.2 itsdangerous-1.1.0
Removing intermediate container 842b3832f45e
 ---> 02d1c5ee79d6
Step 5/6 : COPY . .
 ---> a46dacc41a3d
Step 6/6 : CMD ["python3", "-m", "flask", "run", "--host=0.0.0.0"]
 ---> Running in da976c33fa87
Removing intermediate container da976c33fa87
 ---> 6af1fd62b67b
Successfully built 6af1fd62b67b
Successfully tagged python-docker:latest
```
### View local images
To see a list of images we have on our local machine, we have 2 options. One is to use the CLI and the other is to use [Docker Desktop](https://docs.docker.com/desktop/dashboard/#explore-your-images). As we are currently working in the terminal, let's take a look at listing images using the CLI.

To list images, simply run `docker images` command. You should see at least 2 images listed. One for the base image `3.8-slim-buster` and the other for the image we just built `python-docker:latest`.

### Tag Images
As mentioned earlier, an image name is made up of slash-separated name components. Name components may contain lowercase letters, digits, and separators. A separator is defined as a period, one or two underscores, or one or more dashes. A name component may not start or end with a separator.

An image is made up of a manifest and a list of layers. Do not worry too much about manifests and layers at this point other than a "tag" points to a combination of these artifacts. You can have multiple tags for an image. Let's create a second tag for the image we build and take a look at its layers.

To create a new tag for the image we've built already, run the following command:
```
docker tag python-docker:latest python-docker:v1.0.0
```
The `docker tag` command creates a new tag for an image. it does **NOT** create a new image. The tag points to the same image and is just another way to reference the image.

Now run the `docker images` command to see a list of our local images. You can see that we have two images that start with `python-docker`. We know they are the same image because if you take a look at the `IMAGE ID` column, you can see that the values are the same for the two images.

Let's remove the tag that we just created. To do this, we'll use the `rmi` command. The `rmi` command stands for *remove image*.
```
$ docker rmi python-docker:v1.0.0
Untagged: python-docker:v1.0.0
```
Note that the response from Docker tells us that the image has not been removed but only "untagged". You can check this by running the `docker images` command. Our iamge that was tagged with `:v1.0.0` has been removed, but we still have the `python-docker:latest` tag available on our machine.

## Run Containers
### Overview
Now that we've created our sample application, created a Dockerfile that will produce an image, and created an image using the docker command `docker build`, we can run that image and see if our application is running correctly.

A container is a normal operating system process except that this process is isolated in that it has its own file system, its own networking, and its own isolated process tree separated from the host.

TO run an image inside of a container, we use the `docker run` command. The `docker run` command requires one parameter, which is the name of the image. Let's start our image and make sure it's running correctly.
```
docker run python-docker
```
After running this command, you'll notice that you were not returned to the command prompt. This is because our application it a REST server and runs in a loop waiting for incoming requests without returning control back to the OS until we stop the container.

Let's make a `GET` request to the server using the `curl` command.
```
$ curl localhost:8000
curl: (7) Failed to connect to localhost port 8000: Connection refused
```
As you can see, our `curl` command failed because the connection to our server was refused. This means we were not able to connect to the localhost on `port 8000`. This is expected because our container is run in isolation which includes networking. Let's stop the container and restart with `port 8000` published on our local network.

To stop the container, press `ctrl-c`. This will return you to the terminal prompt.

To publish a port for our container, we'll use the `--publish` flag (`-p` for short) on the `docker run` command. The format of the `--publish` command is `[host port]:[container port]`. So, if we wanted to expose port 8000 inside the container to port 3000 outside the container, we would pass `3000:8000` to the `--publish` flag.

We did not specify a port when running the flask application in the container and the default is 5000. If we want our previous request going to port 8000 to work, we can map the host's port 8000 to the container's port 5000:
```
docker run --publish 8000:5000 python-docker
```
Now let's rerun the curl command from above:
```
$ curl localhost:8000
Hello, Docker!
```
Success! 🎉 We were able to connect to the application running inside of our container on port 8000. Switch back to the terminal where your container is running and you should see the GET request logged to the console.
```
172.17.0.1 - - [15/Mar/2021 01:42:57] "GET / HTTP/1.1" 200 -
```
Press `ctrl-c` to stop the container.

### Run in detached mode
This is great so far, but our sample application is a web server and we don't have to be connected to the conatiner. Docker can run you container in detached mode or in the background. To do this, we can use the `--detach` or `-d` for short. Docker starts your container the same as before but this time will "detach" from the container and return you to the terminal prompt.
```
$ docker run -d -p 8000:5000 python-docker
8128ca689293a240daef647aa916f958c8d624ffab426687eca46a9980bbec77
```
Docker started our container in the background and printed the Container ID on the terminal.

Again, let's make sure that our container is running properly. Run the same curl command from above.
```
$ curl localhost:8000
Hello, Docker!
```

### List Containers
Since we ran our container in the background, how do we know if our container is running or what other containers are running on our machine? Well, we can run the `docker ps` command. Just like on Linux to see a list of processes on your machine, we would run the `ps` command. In the same spirit, we can run the `docker ps` command which displays a list of containers running on our machine.
```
$ docker ps
CONTAINER ID   IMAGE           COMMAND                  CREATED         STATUS         PORTS                    NAMES
8128ca689293   python-docker   "python3 -m flask ru…"   3 minutes ago   Up 3 minutes   0.0.0.0:8000->5000/tcp   objective_elgamal
```
The `docker ps` command provides a bunch of information about our running containers. We can see the container ID, the image running inside the container, the command that was used to start the container, when it was created, the status, ports that are exposed, and the name of the container.

You are probably wondering where the name of our container is coming from. Since we didn't provide a name for the container when we started it, Docker generated a random name. We'll fix this in a minute, but first we need to stop the container. To stop the container, run the `docker stop` command which does just that, stops the container. You need to pass the name of the container or you can use the container ID.
```
$ docker stop 8128ca689293
```
Now rerun the `docker ps` command to see a list of running containers.
```
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

### Stop, Start, and Name Containers
You can start, stop, and restart Docker containers. When we stop a container, it is not removed, but the status is changed to stopped and the process inside the container is stopped. When we ran the `docker ps` command in the previous module, the default output only shows running containers. When we pass the `--all` flag (or `-a` for short), we see all the containers on our machine, irrespective of their start or stop status. 
```
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS               NAMES
8128ca689293   python-docker       "python3 -m flask ru…"   12 minutes ago   Exited (137) 3 minutes ago             objective_elgamal
2b0c1b2a0192   python-docker       "python3 -m flask ru…"   14 minutes ago   Exited (0) 14 minutes ago              stupefied_kapitsa
fdfd4f3fca3b   python-docker       "python3 -m flask ru…"   27 minutes ago   Exited (0) 26 minutes ago              vigilant_kirch
```
You should now see several containers listed. These are containers that we started and stopped but have not been removed.

Let's restart the container that we just stopped. Locate the name or container ID of the container we just stopped, and replace the name of the container below in the restart command.
```
$ docker restart 8128ca689293
```
Now list all the containers again using the `docker ps` command.
```
$ docker ps --all
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS                    NAMES
8128ca689293   python-docker       "python3 -m flask ru…"   16 minutes ago   Up 4 seconds                0.0.0.0:8000->5000/tcp   objective_elgamal
2b0c1b2a0192   python-docker       "python3 -m flask ru…"   19 minutes ago   Exited (0) 19 minutes ago                            stupefied_kapitsa
fdfd4f3fca3b   python-docker       "python3 -m flask ru…"   31 minutes ago   Exited (0) 30 minutes ago                            vigilant_kirch
```
Notice that the container we just restarted has been started in detached mode and has port 8000 exposed. Also, observe the status of the container is "Up X seconds". When you restart a container, it starts with the same flags or commands that it was originally started with.

Now, let's stop and remove all of our containers and take a look at fixing the random naming issue. Stop the container we just started. Find the name or container ID of your running container and replace the name in the command below with the name or container ID on your system.
```
$ docker stop 8128ca689293
8128ca689293
```
Now that all of our containers are stopped, let's remove them. When you remove a container, it is no longer running, nor is it in the stopped status. The process inside the container has bee stopped and the metadata for the container has been removed.

To remove a container, simply run the `docker rm` command passing the container name or container ID. You can pass multiple container names to the command using a single command. Again, replace the container names in the following command with the container names or container IDs from your system.
```
$ docker rm 8128ca689293 2b0c1b2a0192 fdfd4f3fca3b
``` 
Run the `docker ps --all` command again to see that all containers are removed.

Now, let's address the random naming issue. Standard practice is to name your containers for the simple reason that it is easier to identify what is running in the container and what application or service it is associated with.

To name a container, we just need to pass the `--name` flag to the `docker run` command.
```
$ docker run -dp 8000:5000 --name rest-server python-docker
7f73b0465e67c80ff2d0596f42bdadb99d47b4a31de712ea7bcf5f455b1790c5
$ docker ps
CONTAINER ID   IMAGE           COMMAND                  CREATED          STATUS          PORTS                    NAMES
7f73b0465e67   python-docker   "python3 -m flask ru…"   14 seconds ago   Up 13 seconds   0.0.0.0:8000->5000/tcp   rest-server
```
That's better! We can now easily identify our container based on the name.