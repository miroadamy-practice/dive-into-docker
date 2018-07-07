CloudGuru course

https://acloud.guru/course/docker-fundamentals/learn/28bdbdbb-d78b-39c0-e086-368fb4321ef1/f9430aac-e1bd-f91c-2dfc-db882d545667/watch

docker rm `docker ps -aq`

# Same thing

docker run docker.io/library/hello-world
docker run library/hello-world
docker run hello-world



See
https://store.docker.com/images/dotnet?tab=description

## Chapter 6 - Docker in the real world

See ../src/06-docker-in-the-real-world/03-creating-a-dockerfile-part-1/Dockerfile

```
FROM python:2.7-alpine

RUN mkdir /app

WORKDIR /app

COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt

COPY . .

LABEL maintainer="Miro Adamy <miro.adamy@gmail.com>" \
      version="1.0"

CMD flask run --host=0.0.0.0 --port=5000

```

Why not copy . . before: because caching, the requirements.txt seldom changes, the pip install is barely re-run

