# Chapter 6 - Docker in the real world (cont)

See ../src/06-docker-in-the-real-world/13-running-scripts-when-a-container-starts


# Running scripts on start

```
FROM python:2.7-alpine

RUN mkdir /app
WORKDIR /app

COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt

COPY . .

LABEL maintainer="Nick Janetakis <nick.janetakis@gmail.com>" \
      version="1.0"

VOLUME ["/app/public"]

COPY docker-entrypoint.sh /
RUN chmod +x /docker-entrypoint.sh
ENTRYPOINT ["/docker-entrypoint.sh"]

CMD flask run --host=0.0.0.0 --port=5000
```

Special script -  entrypoint.sh

```
#!/bin/sh
set -e

echo "The Dockerfile ENTRYPOINT has been executed!"

export WEB2_COUNTER_MSG="${WEB2_COUNTER_MSG:-carbon based life forms have sensed this website}"

exec "$@"

```

Run

docker container run -itd --rm -p 6379:6379 --name redis --net firstnet -v web2_redis:/data redis:3.2-alpine
docker image build -t webentrypoint .
docker container run -it --rm -p 5000:5000 -e FLASK_APP=app.py -e FLASK_DEBUG=1 -v $PWD:/app --name webentrypoint --net firstnet webentrypoint

Now it returns a message

```
➜  diveintodocker git:(master) ✗ curl localhost:5000
1 carbon based life forms have sensed this website%                                                                                                                                           
➜  diveintodocker git:(master) ✗ curl localhost:5000
2 carbon based life forms have sensed this website%                                                                                                                                           
➜  diveintodocker git:(master) ✗ curl localhost:5000
3 carbon based life forms have sensed this website%
```

Pass new value via ENV VAR and re-run

```
➜  13-running-scripts-when-a-container-starts git:(master) ✗ docker container run -it --rm -p 5000:5000 -e FLASK_APP=app.py -e FLASK_DEBUG=1 -e WEB2_COUNTER_MSG="Docker Rulez" --name webentrypoint --net firstnet webentrypoint
The Dockerfile ENTRYPOINT has been executed!
 * Serving Flask app "app.app"
 * Forcing debug mode on
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 195-037-639
172.20.0.1 - - [08/Jul/2018 10:16:13] "GET / HTTP/1.1" 200 -
172.20.0.1 - - [08/Jul/2018 10:16:14] "GET / HTTP/1.1" 200 -
172.20.0.1 - - [08/Jul/2018 10:16:15] "GET / HTTP/1.1" 200 -
172.20.0.1 - - [08/Jul/2018 10:16:16] "GET / HTTP/1.1" 200 -

----

➜  diveintodocker git:(master) ✗ curl localhost:5000
4 Docker Rulez%                                                                                                                                                                               
➜  diveintodocker git:(master) ✗ curl localhost:5000
5 Docker Rulez%                                                                                                                                                                               
➜  diveintodocker git:(master) ✗ curl localhost:5000
6 Docker Rulez%                                                                                                                                                                               
➜  diveintodocker git:(master) ✗ curl localhost:5000
7 Docker Rulez%

```

## How it is done

Script points

```
#!/bin/sh
set -e

echo "The Dockerfile ENTRYPOINT has been executed!"

export WEB2_COUNTER_MSG="${WEB2_COUNTER_MSG:-carbon based life forms have sensed this website}"

exec "$@"

```

- use sh, not bash (because of alpine)
- set -e => abort on error
- export the variable, defaults to some message
- last exec CRUCIAL

Default entrypoint => /bin/sh -c

Real CMD is

```
/bin/sh -c "flask run --host=0.0.0.0 --port=5000"
```

ENTRYPOINT WITH CMD

```
/bin/sh -c "ENTRYPOINT CMD"
/bin/sh -c "/docker-entrypoint.sh flask run --host=0.0.0.0 --port=5000"

=>

exec "$@" => exec flask run --host=0.0.0.0 --port=5000
```


# Cleanup

docker container ls
docker container ls -a

```
➜  diveintodocker git:(master) ✗ docker system df
TYPE                TOTAL               ACTIVE              SIZE                RECLAIMABLE
Images              42                  0                   16.49GB             16.49GB (100%)
Containers          0                   0                   0B                  0B
Local Volumes       1                   0                   98B                 98B (100%)
Build Cache                                                 0B                  0B

```

Dangling images

```
➜  diveintodocker git:(master) ✗ docker image ls | grep '<none'
<none>                                  <none>              2782b59502eb        19 hours ago        84.3MB

```



