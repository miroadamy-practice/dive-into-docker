# Microservices

Running complex system - src/07-docker-compose-in-the-real-world/06-managing-microservices-with-docker-compose

```


```


Start

```
docker-compose up --build


➜  06-managing-microservices-with-docker-compose git:(master) docker-compose ps
     Name                    Command               State            Ports
----------------------------------------------------------------------------------
voting_db_1       docker-entrypoint.sh postgres    Up      5432/tcp
voting_redis_1    docker-entrypoint.sh redis ...   Up      0.0.0.0:32770->6379/tcp
voting_result_1   nodemon --debug server.js        Up      0.0.0.0:5001->80/tcp
voting_vote_1     python app.py                    Up      0.0.0.0:5000->80/tcp
voting_worker_1   /bin/sh -c dotnet Worker.dll     Up

```