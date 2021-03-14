# Python Docker Sample Application
Source: https://docs.docker.com/language/python/

## Tools Needed for Tutorial
* [Python version 3.8 or later](https://www.python.org/downloads/) or follow [Opensource Python installation](https://opensource.com/article/19/5/python-3-default-mac)
* [Docker running locally](https://docs.docker.com/desktop/)
* [An IDE or a text editor to edit files - VSCode](https://code.visualstudio.com/Download)

## Tutorial Instructions
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