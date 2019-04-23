#Prerequisites

#Make sure you have already installed both Docker Engine and Docker Compose. You don’t need to install Python or Redis, as both are provided by Docker images.

Step 1: Setup

1.
Define the application dependencies.

#    Create a directory for the project:

    $ mkdir composetest
    $ cd composetest

2.
#Create a file called app.py in your project directory and paste this in:

import time

import redis
from flask import Flask


app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)


def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)


@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)

if __name__ == "__main__":
    app.run(host="0.0.0.0", debug=True)

#In this example, redis is the hostname of the redis container on the application’s network. We use the default port for Redis, 6379.

3. 
#Create another file called requirements.txt in your project directory and paste this in:

flask
redis


Step 2: Create a Dockerfile

#In this step, you write a Dockerfile that builds a Docker image. The image contains all the dependencies the Python application requires, including Python itself.

#In your project directory, create a file named Dockerfile and paste the following:

FROM python:3.4-alpine
ADD . /code
WORKDIR /code
RUN pip install -r requirements.txt
CMD ["python", "app.py"]

#This tells Docker to:

#    Build an image starting with the Python 3.4 image.
#    Add the current directory . into the path /code in the image.
#    Set the working directory to /code.
#    Install the Python dependencies.
#    Set the default command for the container to python app.py.

Step 3: Define services in a Compose file

#Create a file called docker-compose.yml in your project directory and paste the following:

version: '3'
services:
  web:
    build: .
    ports:
     - "5000:5000"
  redis:
    image: "redis:alpine"

#This Compose file defines two services, web and redis. The web service:

#   Uses an image that’s built from the Dockerfile in the current directory.
#    Forwards the exposed port 5000 on the container to port 5000 on the host machine. We use the default port for the Flask web server, 5000.

#The redis service uses a public Redis image pulled from the Docker Hub registry.


Step 4: Build and run your app with Compose

1.
#    From your project directory, start up your application by running docker-compose up.

    $ docker-compose up
#    Creating network "composetest_default" with the default driver
#    Creating composetest_web_1 ...
#    Creating composetest_redis_1 ...
#    Creating composetest_web_1
#    Creating composetest_redis_1 ... done
#    Attaching to composetest_web_1, composetest_redis_1
#    web_1    |  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
#    redis_1  | 1:C 17 Aug 22:11:10.480 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
#    redis_1  | 1:C 17 Aug 22:11:10.480 # Redis version=4.0.1, bits=64, commit=00000000, modified=0, pid=1, just started
#    redis_1  | 1:C 17 Aug 22:11:10.480 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
#    web_1    |  * Restarting with stat
#    redis_1  | 1:M 17 Aug 22:11:10.483 * Running mode=standalone, port=6379.
#    redis_1  | 1:M 17 Aug 22:11:10.483 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
#    web_1    |  * Debugger is active!
#    redis_1  | 1:M 17 Aug 22:11:10.483 # Server initialized
#    redis_1  | 1:M 17 Aug 22:11:10.483 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
#    web_1    |  * Debugger PIN: 330-787-903
#    redis_1  | 1:M 17 Aug 22:11:10.483 * Ready to accept connections
#
#    Compose pulls a Redis image, builds an image for your code, and starts the services you defined. In this case, the code is statically copied into the image at build time.

2.
    Enter http://0.0.0.0:5000/ in a browser to see the application running.

#   If you’re using Docker natively on Linux, Docker Desktop for Mac, or Docker Desktop for Windows, then the web app should now be listening on port 5000 on your Docker daemon host. Point your web browser to http://localhost:5000 to find the Hello World message. If this doesn’t resolve, you can also try http://0.0.0.0:5000.

#    If you’re using Docker Machine on a Mac or Windows, use docker-machine ip MACHINE_VM to get the IP address of your Docker host. Then, open http://MACHINE_VM_IP:5000 in a browser.

    You should see a message in your browser saying:

    Hello World! I have been seen 1 times.

#Refresh the page.
3.
 The number should increment.

Hello World! I have been seen 2 times.




If any queries go and open this link:
  https://docs.docker.com/compose/gettingstarted/ 
