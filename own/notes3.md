# Chapter 6 - Docker in the real world (cont)

See ../src/06-docker-in-the-real-world/11-sharing-data-between-containers


New Dockerfile

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

CMD flask run --host=0.0.0.0 --port=5000


```

Created web2_redis

```
➜  11-sharing-data-between-containers git:(master) docker container run -itd --rm -p 5000:5000 -e FLASK_APP=app.py -e FLASK_DEBUG=1 -v $PWD:/app --name web2 --net firstnet web2
cdb27b9e0b268246c299f48fae552bd19e2c87124acb16e36b09763b6c1deac9
➜  11-sharing-data-between-containers git:(master) docker container run -itd --rm -p 6379:6379 --name redis --net firstnet -v web2_redis:/data --volumes-from=web2 redis:3.2-alpine
f62ab0bd965efc818892989eeb3e4247d3b08ee1fdb6119b68d88b66225558fe


```

Now I can see the files from /app/public inside redis

```
➜  11-sharing-data-between-containers git:(master) docker container exec -it redis sh
/data # ls -la /app/
total 28
drwxr-xr-x    9 root     root           288 Jul  7 23:08 .
drwxr-xr-x    1 root     root          4096 Jul  7 23:09 ..
-rwxr-xr-x    1 root     root           288 Apr  1  2017 Dockerfile
-rwxr-xr-x    1 root     root             0 Feb 13  2017 __init__.py
-rw-r--r--    1 root     root           103 Jul  7 23:08 __init__.pyc
-rwxr-xr-x    1 root     root           232 Feb 19  2017 app.py
-rw-r--r--    1 root     root           524 Jul  7 23:08 app.pyc
drwxr-xr-x    2 root     root          4096 Jul  7 23:08 public
-rwxr-xr-x    1 root     root            29 Feb 18  2017 requirements.txt

/data # ls -la /app/public/
total 8
drwxr-xr-x    2 root     root          4096 Jul  7 23:08 .
drwxr-xr-x    9 root     root           288 Jul  7 23:08 ..
-rwxr-xr-x    1 root     root            21 Feb 21  2017 main.css
/data # %                                                                                                                         

```

I will NOT see files created OUTSIDE of containers after they are started

```
➜  11-sharing-data-between-containers git:(master) ✗ touch public/newfile.txt

➜  11-sharing-data-between-containers git:(master) ✗ docker container exec -it redis sh
/data # ls -la /app/public/
total 8
drwxr-xr-x    2 root     root          4096 Jul  7 23:08 .
drwxr-xr-x    9 root     root           288 Jul  7 23:08 ..
-rwxr-xr-x    1 root     root            21 Feb 21  2017 main.css
/data # ls -la /app/public/
total 8
drwxr-xr-x    2 root     root          4096 Jul  7 23:08 .
drwxr-xr-x    9 root     root           288 Jul  7 23:08 ..
-rwxr-xr-x    1 root     root            21 Feb 21  2017 main.css
/data # %                                                                                                                         

➜  11-sharing-data-between-containers git:(master) ✗ ll public
total 8
-rwxr-xr-x  1 miro  staff    21B 21 Feb  2017 main.css
-rw-r--r--  1 miro  staff     0B  8 Jul 01:11 newfile.txt

```

But if I create file from within one container, I can see it in other

```
➜  11-sharing-data-between-containers git:(master) ✗ docker container exec -it web2 sh
/app # ls -la public/
total 8
drwxr-xr-x    2 root     root          4096 Jul  7 23:08 .
drwxr-xr-x    9 root     root           288 Jul  7 23:08 ..
-rwxr-xr-x    1 root     root            21 Feb 21  2017 main.css

/app # touch public/another.txt

/app # ls -la public/
total 8
drwxr-xr-x    2 root     root          4096 Jul  7 23:13 .
drwxr-xr-x    9 root     root           288 Jul  7 23:08 ..
-rw-r--r--    1 root     root             0 Jul  7 23:13 another.txt
-rwxr-xr-x    1 root     root            21 Feb 21  2017 main.css

/app # %                                                                                                                         

➜  11-sharing-data-between-containers git:(master) ✗ docker container exec -it redis sh
/data # ls -la /app/public/
total 8
drwxr-xr-x    2 root     root          4096 Jul  7 23:13 .
drwxr-xr-x    9 root     root           288 Jul  7 23:08 ..
-rw-r--r--    1 root     root             0 Jul  7 23:13 another.txt
-rwxr-xr-x    1 root     root            21 Feb 21  2017 main.css
/data # %                                                                                                                                                                                     
➜  11-sharing-data-between-containers git:(master) ✗

```

Also, files created from WITHIN are not visible OUTSIDE (dockerhost)

```
➜  11-sharing-data-between-containers git:(master) ✗ ls -la public
total 8
drwxr-xr-x  4 miro  staff  128  8 Jul 01:11 .
drwxr-xr-x  9 miro  staff  288  8 Jul 01:08 ..
-rwxr-xr-x  1 miro  staff   21 21 Feb  2017 main.css
-rw-r--r--  1 miro  staff    0  8 Jul 01:11 newfile.txt

```

Same effect as the directive

VOLUME ["/app/public"]

can be achieved by leaving it out and using

```
docker container run -itd --rm -p 5000:5000 -e FLASK_APP=app.py -e FLASK_DEBUG=1 -v $PWD:/app -v /app/public --name web2 --net firstnet web2

docker container run -itd --rm -p 6379:6379 --name redis --net firstnet -v web2_redis:/data --volumes-from=web2 redis:3.2-alpine
```


# Shrinkage

See src/06-docker-in-the-real-world/12-optimizing-your-docker-images



