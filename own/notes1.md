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

```
➜  03-creating-a-dockerfile-part-1 git:(master) ✗ docker image build -t web1 .
Sending build context to Docker daemon  5.632kB
Step 1/8 : FROM python:2.7-alpine
2.7-alpine: Pulling from library/python
911c6d0c7995: Pull complete
17a85618b748: Pull complete
880358d57a6c: Pull complete
eeaf1c636ed0: Pull complete
Digest: sha256:78cacea08bf113aa6799301591939c29580357d3c786461a89180616a2c8c0d6
Status: Downloaded newer image for python:2.7-alpine
 ---> c57ed7d143f9
Step 2/8 : RUN mkdir /app
 ---> Running in c0a04851a536
Removing intermediate container c0a04851a536
 ---> d682e10a7dea
Step 3/8 : WORKDIR /app
Removing intermediate container 80717384378f
 ---> b68717c622fe
Step 4/8 : COPY requirements.txt requirements.txt
 ---> a87623ad7d3b
Step 5/8 : RUN pip install -r requirements.txt
 ---> Running in 59bec2a8bddb
Collecting Flask==0.12 (from -r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/0e/e9/37ee66dde483dceefe45bb5e92b387f990d4f097df40c400cf816dcebaa4/Flask-0.12-py2.py3-none-any.whl (82kB)
Collecting itsdangerous>=0.21 (from Flask==0.12->-r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/dc/b4/a60bcdba945c00f6d608d8975131ab3f25b22f2bcfe1dab221165194b2d4/itsdangerous-0.24.tar.gz (46kB)
Collecting click>=2.0 (from Flask==0.12->-r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/34/c1/8806f99713ddb993c5366c362b2f908f18269f8d792aff1abfd700775a77/click-6.7-py2.py3-none-any.whl (71kB)
Collecting Jinja2>=2.4 (from Flask==0.12->-r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/7f/ff/ae64bacdfc95f27a016a7bed8e8686763ba4d277a78ca76f32659220a731/Jinja2-2.10-py2.py3-none-any.whl (126kB)
Collecting Werkzeug>=0.7 (from Flask==0.12->-r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/20/c4/12e3e56473e52375aa29c4764e70d1b8f3efa6682bef8d0aae04fe335243/Werkzeug-0.14.1-py2.py3-none-any.whl (322kB)
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
Installing collected packages: itsdangerous, click, MarkupSafe, Jinja2, Werkzeug, Flask
Successfully installed Flask-0.12 Jinja2-2.10 MarkupSafe-1.0 Werkzeug-0.14.1 click-6.7 itsdangerous-0.24
Removing intermediate container 59bec2a8bddb
 ---> 0e8d97cfeca8
Step 6/8 : COPY . .
 ---> 1db43f0bf122
Step 7/8 : LABEL maintainer="Miro Adamy <miro.adamy@gmail.com>"       version="1.0"
 ---> Running in c3c15d27ceaf
Removing intermediate container c3c15d27ceaf
 ---> 26dd7420026f
Step 8/8 : CMD flask run --host=0.0.0.0 --port=5000
 ---> Running in 6fd02ee0d8bd
Removing intermediate container 6fd02ee0d8bd
 ---> e3475b575732
Successfully built e3475b575732
Successfully tagged web1:latest
```

Result

```
➜  03-creating-a-dockerfile-part-1 git:(master) ✗ docker images | grep web1
web1                                                             latest              e3475b575732        52 seconds ago      83.7MB
```

### Re-run

All cached

```
➜  03-creating-a-dockerfile-part-1 git:(master) ✗ docker image build -t web1 .
Sending build context to Docker daemon  5.632kB
Step 1/8 : FROM python:2.7-alpine
 ---> c57ed7d143f9
Step 2/8 : RUN mkdir /app
 ---> Using cache
 ---> d682e10a7dea
Step 3/8 : WORKDIR /app
 ---> Using cache
 ---> b68717c622fe
Step 4/8 : COPY requirements.txt requirements.txt
 ---> Using cache
 ---> a87623ad7d3b
Step 5/8 : RUN pip install -r requirements.txt
 ---> Using cache
 ---> 0e8d97cfeca8
Step 6/8 : COPY . .
 ---> Using cache
 ---> 1db43f0bf122
Step 7/8 : LABEL maintainer="Miro Adamy <miro.adamy@gmail.com>"       version="1.0"
 ---> Using cache
 ---> 26dd7420026f
Step 8/8 : CMD flask run --host=0.0.0.0 --port=5000
 ---> Using cache
 ---> e3475b575732
Successfully built e3475b575732
Successfully tagged web1:latest
```

Change maintainer and re-run - build from COPY down

```
➜  03-creating-a-dockerfile-part-1 git:(master) docker image build -t web1 .
Sending build context to Docker daemon  5.632kB
Step 1/8 : FROM python:2.7-alpine
 ---> c57ed7d143f9
Step 2/8 : RUN mkdir /app
 ---> Using cache
 ---> d682e10a7dea
Step 3/8 : WORKDIR /app
 ---> Using cache
 ---> b68717c622fe
Step 4/8 : COPY requirements.txt requirements.txt
 ---> Using cache
 ---> a87623ad7d3b
Step 5/8 : RUN pip install -r requirements.txt
 ---> Using cache
 ---> 0e8d97cfeca8
Step 6/8 : COPY . .
 ---> d718352071f2
Step 7/8 : LABEL maintainer="Miroslav Adamy <miro.adamy@gmail.com>"       version="1.0"
 ---> Running in ac1156beeb10
Removing intermediate container ac1156beeb10
 ---> e471d34fb3b7
Step 8/8 : CMD flask run --host=0.0.0.0 --port=5000
 ---> Running in 56cf8d612ef5
Removing intermediate container 56cf8d612ef5
 ---> bbe2a732d606
Successfully built bbe2a732d606
Successfully tagged web1:latest
```

## Inspect

```
➜  03-creating-a-dockerfile-part-1 git:(master) ✗ docker image inspect web1
[
    {
        "Id": "sha256:bbe2a732d606b849e7080548879b89c39744a1032888eb92bedbaf672b24bbf9",
        "RepoTags": [
            "web1:latest"
        ],
        "RepoDigests": [],
        "Parent": "sha256:e471d34fb3b7ea373ba8bd165c44cb102a1da07b81797c592e56b3d0aa6922a8",
        "Comment": "",
        "Created": "2018-07-07T13:19:06.2069216Z",
        "Container": "56cf8d612ef5b958249059944e979bcbed98bd760cb2d816bca80c75b9b8e81d",
        "ContainerConfig": {
            "Hostname": "56cf8d612ef5",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "LANG=C.UTF-8",
                "PYTHONIOENCODING=UTF-8",
                "GPG_KEY=C01E1CAD5EA2C4F0B8E3571504C367C218ADD4FF",
                "PYTHON_VERSION=2.7.15",
                "PYTHON_PIP_VERSION=10.0.1"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "#(nop) ",
                "CMD [\"/bin/sh\" \"-c\" \"flask run --host=0.0.0.0 --port=5000\"]"
            ],
            "ArgsEscaped": true,
            "Image": "sha256:e471d34fb3b7ea373ba8bd165c44cb102a1da07b81797c592e56b3d0aa6922a8",
            "Volumes": null,
            "WorkingDir": "/app",
            "Entrypoint": null,
            "OnBuild": [],
            "Labels": {
                "maintainer": "Miroslav Adamy <miro.adamy@gmail.com>",
                "version": "1.0"
            }
        },
        "DockerVersion": "18.03.1-ce",
        "Author": "",
        "Config": {
            "Hostname": "",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "LANG=C.UTF-8",
                "PYTHONIOENCODING=UTF-8",
                "GPG_KEY=C01E1CAD5EA2C4F0B8E3571504C367C218ADD4FF",
                "PYTHON_VERSION=2.7.15",
                "PYTHON_PIP_VERSION=10.0.1"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "flask run --host=0.0.0.0 --port=5000"
            ],
            "ArgsEscaped": true,
            "Image": "sha256:e471d34fb3b7ea373ba8bd165c44cb102a1da07b81797c592e56b3d0aa6922a8",
            "Volumes": null,
            "WorkingDir": "/app",
            "Entrypoint": null,
            "OnBuild": [],
            "Labels": {
                "maintainer": "Miroslav Adamy <miro.adamy@gmail.com>",
                "version": "1.0"
            }
        },
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 83745186,
        "VirtualSize": 83745186,
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/dafb2bce711fd4af226f1a2a473441f2594ab9fabe03b6d2454a4ee107e990df/diff:/var/lib/docker/overlay2/8b437c68368f26ff42a39a06c36a438af6538860ea7dc0c7362f8750e497ec3b/diff:/var/lib/docker/overlay2/a9f27971d63c91dcb74d5cc5933983f2c521b3a7033508e8882eed1e3dbf6ac5/diff:/var/lib/docker/overlay2/30b23adab56edfed4af35094fbdd4d04da295f3f348ca690d3ec73612bdfae6d/diff:/var/lib/docker/overlay2/e664856fe5903ea552c461b8a9a5fdfef7e891c0428e1a47332b415c9f8ce13a/diff:/var/lib/docker/overlay2/38d398bf2b60d3379b23eaf598edbaa7f9c6a1810df2c595a5b8028cbd5df9c3/diff:/var/lib/docker/overlay2/c5fa2879162d3aa35ac8a705982356b00377d5569ec9bbf6878651d0ae5185a0/diff",
                "MergedDir": "/var/lib/docker/overlay2/23c56f6ec90e76ca0d893d941ca0d29b81bd18c40b462c9977f5508c4964cec9/merged",
                "UpperDir": "/var/lib/docker/overlay2/23c56f6ec90e76ca0d893d941ca0d29b81bd18c40b462c9977f5508c4964cec9/diff",
                "WorkDir": "/var/lib/docker/overlay2/23c56f6ec90e76ca0d893d941ca0d29b81bd18c40b462c9977f5508c4964cec9/work"
            },
            "Name": "overlay2"
        },
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:717b092b8c86356cf976d9c83fa6f0ea67f2bf3148a5bbb7e02026a5d3245e68",
                "sha256:2df3095bdb63457521100a3ec9ae8bd9da8d75078a81d7df11b7988d3a98a0a1",
                "sha256:9883b6d82ddfc1f002ff4b98c6198a7ed758ef0cf9631f2507e9c89ea544ae2c",
                "sha256:176be293f240789763ad1304b5b629d80e0a6a6b69ab32c1192125c4b420fb71",
                "sha256:243405369aefab4e4306d506c3a9ba0997ebd4cdfcc6f1b5653cb57e608d05bf",
                "sha256:de3d036e5bf59612a688357ab5f3042dc2e047f10dcc49e34122b3c0a45d4d7c",
                "sha256:91009e9c92fdb315cf7531375b2b96a4d8abfdc5798950571a93626af1b69fc2",
                "sha256:be5d60188f6fba682c86f544f6ccc75098be72c4117e95cf0914f46dc87b3a9b"
            ]
        },
        "Metadata": {
            "LastTagTime": "2018-07-07T13:19:06.267386Z"
        }
    }
]
```
