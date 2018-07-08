# Docker compose 1

See src/07-docker-compose-in-the-real-world/03-adding-docker-compose-support-to-our-web-app


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
