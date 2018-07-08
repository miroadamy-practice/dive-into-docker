# Docker compose 1

See src/07-docker-compose-in-the-real-world/03-adding-docker-compose-support-to-our-web-app

Good example of scripting in https://github.com/docker-library/postgres/blob/master/10/docker-entrypoint.sh
- how to use VAR from file



If using both build and image, image defines the name

```
version: '3'

services:
  redis:
    image: 'redis:3.2-alpine'
    ports:
      - '6379:6379'
    volumes:
      - 'redis:/data'

  web:
    build:  '.'
    image:  'miroadamy/web:1.0-comp'
    depends_on:
      - 'redis'
    env_file:
      - '.env'  
    ports:
      - '5000:5000'

    volumes:
    - '.:/app'
      
volumes:
  redis: {}

```

The .env file

```
COMPOSE_PROJECT_NAME=web2

PYTHONBUFFERED=true
FLASK_APP=app.py
FLASK_DEBUG=1

```

COMPOSE_PROJECT_NAME - important for repeating subdirs names

## Build

runs only build commands

```
➜  03-adding-docker-compose-support-to-our-web-app git:(master) ✗ docker-compose build
redis uses an image, skipping
Building web
Step 1/12 : FROM python:2.7-alpine
 ---> c57ed7d143f9
Step 2/12 : RUN mkdir /app
 ---> Running in 3fec084705a6
Removing intermediate container 3fec084705a6
 ---> dae60c83b896
Step 3/12 : WORKDIR /app
Removing intermediate container a48cf898bf29
 ---> 10388263dcff
Step 4/12 : COPY requirements.txt requirements.txt
 ---> e4e2d382203f
Step 5/12 : RUN pip install -r requirements.txt
 ---> Running in 864c7667e7a5
Collecting Flask==0.12 (from -r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/0e/e9/37ee66dde483dceefe45bb5e92b387f990d4f097df40c400cf816dcebaa4/Flask-0.12-py2.py3-none-any.whl (82kB)
Collecting flask-redis==0.3 (from -r requirements.txt (line 2))
  Downloading https://files.pythonhosted.org/packages/8b/59/e29f607475ca6ae21e30ff5e6ca0f2fd58701879a31ffeff7471ea3865f6/Flask_Redis-0.3.0-py2.py3-none-any.whl
Collecting itsdangerous>=0.21 (from Flask==0.12->-r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/dc/b4/a60bcdba945c00f6d608d8975131ab3f25b22f2bcfe1dab221165194b2d4/itsdangerous-0.24.tar.gz (46kB)
Collecting click>=2.0 (from Flask==0.12->-r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/34/c1/8806f99713ddb993c5366c362b2f908f18269f8d792aff1abfd700775a77/click-6.7-py2.py3-none-any.whl (71kB)
Collecting Jinja2>=2.4 (from Flask==0.12->-r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/7f/ff/ae64bacdfc95f27a016a7bed8e8686763ba4d277a78ca76f32659220a731/Jinja2-2.10-py2.py3-none-any.whl (126kB)
Collecting Werkzeug>=0.7 (from Flask==0.12->-r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/20/c4/12e3e56473e52375aa29c4764e70d1b8f3efa6682bef8d0aae04fe335243/Werkzeug-0.14.1-py2.py3-none-any.whl (322kB)
Collecting redis>=2.7.6 (from flask-redis==0.3->-r requirements.txt (line 2))
  Downloading https://files.pythonhosted.org/packages/3b/f6/7a76333cf0b9251ecf49efff635015171843d9b977e4ffcf59f9c4428052/redis-2.10.6-py2.py3-none-any.whl (64kB)
Collecting MarkupSafe>=0.23 (from Jinja2>=2.4->Flask==0.12->-r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/4d/de/32d741db316d8fdb7680822dd37001ef7a448255de9699ab4bfcbdf4172b/MarkupSafe-1.0.tar.gz
Building wheels for collected packages: itsdangerous, MarkupSafe
  Running setup.py bdist_wheel for itsdangerous: started
  Running setup.py bdist_wheel for itsdangerous: finished with status 'done'
  Stored in directory: /root/.cache/pip/wheels/2c/4a/61/5599631c1554768c6290b08c02c72d7317910374ca602ff1e5
  Running setup.py bdist_wheel for MarkupSafe: started
  Running setup.py bdist_wheel for MarkupSafe: finished with status 'done'
  Stored in directory: /root/.cache/pip/wheels/33/56/20/ebe49a5c612fffe1c5a632146b16596f9e64676768661e4e46
Successfully built itsdangerous MarkupSafe
Installing collected packages: itsdangerous, click, MarkupSafe, Jinja2, Werkzeug, Flask, redis, flask-redis
Successfully installed Flask-0.12 Jinja2-2.10 MarkupSafe-1.0 Werkzeug-0.14.1 click-6.7 flask-redis-0.3.0 itsdangerous-0.24 redis-2.10.6
Removing intermediate container 864c7667e7a5
 ---> 91359404ad33
Step 6/12 : COPY . .
 ---> 8623fab0d972
Step 7/12 : LABEL maintainer="Nick Janetakis <nick.janetakis@gmail.com>"       version="2.0"
 ---> Running in d3fa48f7cf34
Removing intermediate container d3fa48f7cf34
 ---> a43fa4ee693b
Step 8/12 : VOLUME ["/app/public"]
 ---> Running in 9264bdc7b596
Removing intermediate container 9264bdc7b596
 ---> 7567c8af0d33
Step 9/12 : COPY docker-entrypoint.sh /
 ---> 214ed762c4fc
Step 10/12 : RUN chmod +x /docker-entrypoint.sh
 ---> Running in 0b0324e87b9d
Removing intermediate container 0b0324e87b9d
 ---> 5aedb0f2e51f
Step 11/12 : ENTRYPOINT ["/docker-entrypoint.sh"]
 ---> Running in cd2ee83ca042
Removing intermediate container cd2ee83ca042
 ---> ca5b77cb5982
Step 12/12 : CMD flask run --host=0.0.0.0 --port=5000
 ---> Running in e0ef6f8be736
Removing intermediate container e0ef6f8be736
 ---> fe0731bb669d
Successfully built fe0731bb669d
Successfully tagged miroadamy/web:1.0-comp

➜  03-adding-docker-compose-support-to-our-web-app git:(master) ✗ docker image ls | grep web
miroadamy/web                           1.0-comp            fe0731bb669d        15 seconds ago      84.3MB

```

Without explicit name - prefix is determined by COMPOSE_PROJECT_NAME=web2

```
 web:
    build:  '.'
    # Use this to force name image
    # image:  'miroadamy/web:1.0-comp'

....


➜  03-adding-docker-compose-support-to-our-web-app git:(master) ✗ docker-compose build
redis uses an image, skipping
Building web
Step 1/12 : FROM python:2.7-alpine
... DELETED ...
Removing intermediate container 270a41875e00
 ---> b10472009f19
Successfully built b10472009f19
Successfully tagged web2_web:latest



➜  03-adding-docker-compose-support-to-our-web-app git:(master) ✗ docker image ls | grep web
web2_web                                latest              b10472009f19        4 minutes ago       84.3MB

```

# Pull

- was already there

```
➜  03-adding-docker-compose-support-to-our-web-app git:(master) ✗ docker-compose pull
Pulling redis ... done
Pulling web   ... done
```


# Running



```
➜  03-adding-docker-compose-support-to-our-web-app git:(master) ✗ docker-compose up
Creating network "web2_default" with the default driver
Creating web2_redis_1 ... done
Creating web2_web_1   ... done
Attaching to web2_redis_1, web2_web_1
redis_1  | 1:C 08 Jul 16:53:24.060 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
redis_1  |                 _._
redis_1  |            _.-``__ ''-._
redis_1  |       _.-``    `.  `_.  ''-._           Redis 3.2.12 (00000000/0) 64 bit
redis_1  |   .-`` .-```.  ```\/    _.,_ ''-._
redis_1  |  (    '      ,       .-`  | `,    )     Running in standalone mode
redis_1  |  |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
redis_1  |  |    `-._   `._    /     _.-'    |     PID: 1
redis_1  |   `-._    `-._  `-./  _.-'    _.-'
redis_1  |  |`-._`-._    `-.__.-'    _.-'_.-'|
redis_1  |  |    `-._`-._        _.-'_.-'    |           http://redis.io
redis_1  |   `-._    `-._`-.__.-'_.-'    _.-'
redis_1  |  |`-._`-._    `-.__.-'    _.-'_.-'|
redis_1  |  |    `-._`-._        _.-'_.-'    |
redis_1  |   `-._    `-._`-.__.-'_.-'    _.-'
redis_1  |       `-._    `-.__.-'    _.-'
redis_1  |           `-._        _.-'
redis_1  |               `-.__.-'
redis_1  |
redis_1  | 1:M 08 Jul 16:53:24.061 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
redis_1  | 1:M 08 Jul 16:53:24.061 # Server started, Redis version 3.2.12
redis_1  | 1:M 08 Jul 16:53:24.061 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
redis_1  | 1:M 08 Jul 16:53:24.062 * DB loaded from disk: 0.001 seconds
redis_1  | 1:M 08 Jul 16:53:24.062 * The server is now ready to accept connections on port 6379
web_1    | The Dockerfile ENTRYPOINT has been executed!
web_1    |  * Serving Flask app "app.app"
web_1    |  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
web_1    |  * Restarting with stat
web_1    |  * Debugger is active!
web_1    |  * Debugger PIN: 274-384-099
```

Keeps using same volumes
```
➜  diveintodocker git:(master) ✗ curl localhost:5000
8 carbon based life forms have sensed this website%                                                                                                                                           
➜  diveintodocker git:(master) ✗ curl localhost:5000
9 carbon based life forms have sensed this website%                                                                                                                                           
➜  diveintodocker git:(master) ✗ curl localhost:5000
10 carbon based life forms have sensed this website%                                                                                                                                          
➜  diveintodocker git:(master) ✗ curl localhost:5000
11 carbon based life forms have sensed this website%
```


