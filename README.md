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
To test that the application is working properly, open a new browser and navigate to `http://localhost:5000`. Looking at the terminal where the server is running, you should see similar requests in the server logs:
```
127.0.0.1 - - [14/Mar/2021 15:33:12] "GET / HTTP/1.1" 200 -
```

### Create a Dockerfile for Python
