### Docker Compose

Till now we've spent all our time exploring the Docker client. In the Docker ecosystem, however, there are a bunch of other open-source tools which play very nicely with Docker. A few of them are -

1. [Docker Machine](https://docs.docker.com/machine/) - Create Docker hosts on your computer, on cloud providers, and inside your own data center
2. [Docker Compose](https://docs.docker.com/compose/) - A tool for defining and running multi-container Docker applications.
3. [Docker Swarm](https://docs.docker.com/swarm/) - A native clustering solution for Docker
4. [Kubernetes](https://kubernetes.io) - Kubernetes is an open-source system for automating deployment, scaling, and management of containerized applications.

In this section, we are going to look at one of these tools, Docker Compose, and see how it can make dealing with multi-container apps easier.

The background story of Docker Compose is quite interesting. Roughly two years ago, a company called OrchardUp launched a tool called Fig. The idea behind Fig was to make isolated development environments work with Docker. The project was very well received on [Hacker News](https://news.ycombinator.com/item?id=7132044) - I oddly remember reading about it but didn't quite get the hang of it.

The [first comment](https://news.ycombinator.com/item?id=7133449) on the forum actually does a good job of explaining what Fig is all about.

> So really at this point, that's what Docker is about: running processes. Now Docker offers a quite rich API to run the processes: shared volumes (directories) between containers (i.e. running images), forward port from the host to the container, display logs, and so on.  But that's it: Docker as of now, remains at the process level.

> While it provides options to orchestrate multiple containers to create a single "app", it doesn't address the managemement of such group of containers as a single entity.
> And that's where tools such as Fig come in: talking about a group of containers as a single entity. Think "run an app" (i.e. "run an orchestrated cluster of containers") instead of "run a container".

It turns out that a lot of people using docker agree with this sentiment. Slowly and steadily as Fig became popular, Docker Inc. took notice, acquired the company and re-branded Fig as Docker Compose.

So what is *Compose* used for? Compose is a tool that is used for defining and running multi-container Docker apps in an easy way. It provides a configuration file called `docker-compose.yml` that can be used to bring up an application and the suite of services it depends on with just one command. Compose works in all environments: production, staging, development, testing, as well as CI workflows, although Compose is ideal for development and testing environments.

Let's see if we can create a `docker-compose.yml` file for our SF-Foodtrucks app and evaluate whether Docker Compose lives up to its promise.

The first step, however, is to install Docker Compose. If you're running Windows or Mac, Docker Compose is already installed as it comes in the Docker Toolbox. Linux users can easily get their hands on Docker Compose by following the [instructions](https://docs.docker.com/compose/install/) on the docs. Since Compose is written in Python, you can also simply do the following: 

```bash
sudo pip install docker-compose
```

Test your installation with -

```bash
$ docker-compose --version
docker-compose version 1.21.2, build a133471
```

Now that we have it installed, we can jump on the next step i.e. the Docker Compose file `docker-compose.yml`. The syntax for YAML is quite simple and the repo already contains the docker-compose [file](https://github.com/prakhar1989/FoodTrucks/blob/master/docker-compose.yml) that we'll be using.

```yaml
version: "3"
services:
  es:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.3.2
    container_name: es
    environment:
      - discovery.type=single-node
    ports:
      - 9200:9200
    volumes:
      - esdata1:/usr/share/elasticsearch/data
  web:
    image: prakhar1989/foodtrucks-web
    command: python app.py
    depends_on:
      - es
    ports:
      - 5000:5000
    volumes:
      - .:/opt/flask-app
volumes:
    esdata1:
      driver: local
```

Let me breakdown what the file above means. At the parent level, we define the names of our services - `es` and `web`. For each service, that Docker needs to run, we can add additional parameters out of which `image` is required. For `es`, we just refer to the `elasticsearch` image available on Elastic registry. For our Flask app, we refer to the image that we built at the beginning of this section.

Via other parameters such as `command` and `ports` we provide more information about the container. The `volumes` parameter specifies a mount point in our `web` container where the code will reside. This is purely optional and is useful if you need access to logs etc. We'll later see how this can be useful during development. Refer to the [online reference](https://docs.docker.com/compose/compose-file) to learn more about the parameters this file supports. We also add volumes for `es` container so that the data we load persists between restarts. We also specify `depends_on`, which tells docker to start the `es` container before `web`. You can read more about it on [docker compose docs](https://docs.docker.com/compose/compose-file/#depends_on).

> Note: You must be inside the directory with the `docker-compose.yml` file in order to execute most Compose commands.

Great! Now the file is ready, let's see `docker-compose` in action. But before we start, we need to make sure the ports are free. So if you have the Flask and ES containers running, lets turn them off.

```bash
$ docker stop $(docker ps -q)
39a2f5df14ef
2a1b77e066e6
```

Now we can run `docker-compose`. Navigate to the food trucks directory and run `docker-compose up`.

```bash
$ docker-compose up
Creating network "41-dockercompose_default" with the default driver
Creating foodtrucks_es_1
Creating foodtrucks_web_1
Attaching to foodtrucks_es_1, foodtrucks_web_1
es_1  | [2016-01-11 03:43:50,300][INFO ][node                     ] [Comet] version[2.1.1], pid[1], build[40e2c53/2015-12-15T13:05:55Z]
es_1  | [2016-01-11 03:43:50,307][INFO ][node                     ] [Comet] initializing ...
es_1  | [2016-01-11 03:43:50,366][INFO ][plugins                  ] [Comet] loaded [], sites []
es_1  | [2016-01-11 03:43:50,421][INFO ][env                      ] [Comet] using [1] data paths, mounts [[/usr/share/elasticsearch/data (/dev/sda1)]], net usable_space [16gb], net total_space [18.1gb], spins? [possibly], types [ext4]
es_1  | [2016-01-11 03:43:52,626][INFO ][node                     ] [Comet] initialized
es_1  | [2016-01-11 03:43:52,632][INFO ][node                     ] [Comet] starting ...
es_1  | [2016-01-11 03:43:52,703][WARN ][common.network           ] [Comet] publish address: {0.0.0.0} is a wildcard address, falling back to first non-loopback: {172.17.0.2}
es_1  | [2016-01-11 03:43:52,704][INFO ][transport                ] [Comet] publish_address {172.17.0.2:9300}, bound_addresses {[::]:9300}
es_1  | [2016-01-11 03:43:52,721][INFO ][discovery                ] [Comet] elasticsearch/cEk4s7pdQ-evRc9MqS2wqw
es_1  | [2016-01-11 03:43:55,785][INFO ][cluster.service          ] [Comet] new_master {Comet}{cEk4s7pdQ-evRc9MqS2wqw}{172.17.0.2}{172.17.0.2:9300}, reason: zen-disco-join(elected_as_master, [0] joins received)
es_1  | [2016-01-11 03:43:55,818][WARN ][common.network           ] [Comet] publish address: {0.0.0.0} is a wildcard address, falling back to first non-loopback: {172.17.0.2}
es_1  | [2016-01-11 03:43:55,819][INFO ][http                     ] [Comet] publish_address {172.17.0.2:9200}, bound_addresses {[::]:9200}
es_1  | [2016-01-11 03:43:55,819][INFO ][node                     ] [Comet] started
es_1  | [2016-01-11 03:43:55,826][INFO ][gateway                  ] [Comet] recovered [0] indices into cluster_state
es_1  | [2016-01-11 03:44:01,825][INFO ][cluster.metadata         ] [Comet] [sfdata] creating index, cause [auto(index api)], templates [], shards [5]/[1], mappings [truck]
es_1  | [2016-01-11 03:44:02,373][INFO ][cluster.metadata         ] [Comet] [sfdata] update_mapping [truck]
es_1  | [2016-01-11 03:44:02,510][INFO ][cluster.metadata         ] [Comet] [sfdata] update_mapping [truck]
es_1  | [2016-01-11 03:44:02,593][INFO ][cluster.metadata         ] [Comet] [sfdata] update_mapping [truck]
es_1  | [2016-01-11 03:44:02,708][INFO ][cluster.metadata         ] [Comet] [sfdata] update_mapping [truck]
es_1  | [2016-01-11 03:44:03,047][INFO ][cluster.metadata         ] [Comet] [sfdata] update_mapping [truck]
web_1 |  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
```

Head over to the IP to see your app live. That was amazing wasn't it? Just few lines of configuration and we have two Docker containers running successfully in unison. Let's stop the services and re-run in detached mode.

```bash
web_1 |  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
Killing foodtrucks_web_1 ... done
Killing foodtrucks_es_1 ... done

$ docker-compose up -d
Creating es               ... done
Creating foodtrucks_web_1 ... done

$ docker-compose ps
      Name                    Command               State                Ports
--------------------------------------------------------------------------------------------
es                 /usr/local/bin/docker-entr ...   Up      0.0.0.0:9200->9200/tcp, 9300/tcp
foodtrucks_web_1   python app.py                    Up      0.0.0.0:5000->5000/tcp
```

Unsurprisingly, we can see both the containers running successfully. Where do the names come from? Those were created automatically by Compose. But does *Compose* also create the network automatically? Good question! Let's find out.

First off, let us stop the services from running. We can always bring them back up in just one command. Data volumes will persist, so it’s possible to start the cluster again with the same data using docker-compose up. To destroy the cluster and the data volumes, just type `docker-compose down -v`.

```bash
$ docker-compose down -v
Stopping foodtrucks_web_1 ... done
Stopping es               ... done
Removing foodtrucks_web_1 ... done
Removing es               ... done
Removing network 41-dockercompose_default
Removing volume 41-dockercompose_default
```

While we're are at it, we'll also remove the `foodtrucks` network that we created last time.

```bash
$ docker network rm 41-dockercompose_default
$ docker network ls
NETWORK ID          NAME                 DRIVER              SCOPE
c2c695315b3a        bridge               bridge              local
a875bec5d6fd        host                 host                local
ead0e804a67b        none                 null                local
```

Great! Now that we have a clean slate, let's re-run our services and see if *Compose* does it's magic.

```bash
$ docker-compose up -d
Recreating foodtrucks_es_1
Recreating foodtrucks_web_1

$ docker container ls
CONTAINER ID        IMAGE                        COMMAND                  CREATED             STATUS              PORTS                    NAMES
f50bb33a3242        prakhar1989/foodtrucks-web   "python app.py"          14 seconds ago      Up 13 seconds       0.0.0.0:5000->5000/tcp   foodtrucks_web_1
e299ceeb4caa        elasticsearch                "/docker-entrypoint.s"   14 seconds ago      Up 14 seconds       9200/tcp, 9300/tcp       foodtrucks_es_1
```

So far, so good. Time to see if any networks were created.

```bash
$ docker network ls
NETWORK ID          NAME                       DRIVER
c2c695315b3a        bridge                     bridge              local
f3b80f381ed3        41-dockercompose_default   bridge              local
a875bec5d6fd        host                       host                local
ead0e804a67b        none                       null                local
```

You can see that compose went ahead and created a new network called `foodtrucks_default` and attached both the new services in that network so that each of these are discoverable to the other. Each container for a service joins the default network and is both reachable by other containers on that network, and discoverable by them at a hostname identical to the container name. 

```bash
$ docker ps
CONTAINER ID        IMAGE                                                 COMMAND                  CREATED              STATUS              PORTS                              NAMES
8c6bb7e818ec        docker.elastic.co/elasticsearch/elasticsearch:6.3.2   "/usr/local/bin/dock…"   About a minute ago   Up About a minute   0.0.0.0:9200->9200/tcp, 9300/tcp   es
7640cec7feb7        prakhar1989/foodtrucks-web                            "python app.py"          About a minute ago   Up About a minute   0.0.0.0:5000->5000/tcp             foodtrucks_web_1

$ docker network inspect 41-dockercompose_default
[
    {
        "Name": "41-dockercompose_default",
        "Id": "f3b80f381ed3e03b3d5e605e42c4a576e32d38ba24399e963d7dad848b3b4fe7",
        "Created": "2018-07-30T03:36:06.0384826Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.19.0.0/16",
                    "Gateway": "172.19.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": true,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "7640cec7feb7f5615eaac376271a93fb8bab2ce54c7257256bf16716e05c65a5": {
                "Name": "foodtrucks_web_1",
                "EndpointID": "b1aa3e735402abafea3edfbba605eb4617f81d94f1b5f8fcc566a874660a0266",
                "MacAddress": "02:42:ac:13:00:02",
                "IPv4Address": "172.19.0.2/16",
                "IPv6Address": ""
            },
            "8c6bb7e818ec1f88c37f375c18f00beb030b31f4b10aee5a0952aad753314b57": {
                "Name": "es",
                "EndpointID": "649b3567d38e5e6f03fa6c004a4302508c14a5f2ac086ee6dcf13ddef936de7b",
                "MacAddress": "02:42:ac:13:00:03",
                "IPv4Address": "172.19.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {
            "com.docker.compose.network": "default",
            "com.docker.compose.project": "foodtrucks",
            "com.docker.compose.version": "1.21.2"
        }
    }
]
```

### Development Workflow

Before we jump to the next section, there's one last thing I wanted to cover about docker-compose. As stated earlier, docker-compose is really great for development and testing. So let's see how we can configure compose to make our lives easier during development.

Throughout this tutorial, we've worked with readymade docker images. While we've built images from scratch, we haven't touched any application code yet and mostly restricted ourselves to editing Dockerfiles and YAML configurations. One thing that you must be wondering is how does the workflow look during development? Is one supposed to keep creating Docker images for every change, then publish it and then run it to see if the changes works as expected? I'm sure that sounds super tedious. There has to be a better way. In this section, that's what we're going to explore.

Let's see how we can make a change in the Foodtrucks app we just ran. Make sure you have the app running,

```bash
$ docker container ls
CONTAINER ID        IMAGE                                                 COMMAND                  CREATED             STATUS              PORTS                              NAMES
5450ebedd03c        prakhar1989/foodtrucks-web                            "python app.py"          9 seconds ago       Up 6 seconds        0.0.0.0:5000->5000/tcp             foodtrucks_web_1
05d408b25dfe        docker.elastic.co/elasticsearch/elasticsearch:6.3.2   "/usr/local/bin/dock…"   10 hours ago        Up 10 hours         0.0.0.0:9200->9200/tcp, 9300/tcp   es
```

Now let's see if we can change this app to display a `Hello world!` message when a request is made to `/hello` route. Currently, the app responds with a 404.

```bash
$ curl -I 0.0.0.0:5000/hello
HTTP/1.0 404 NOT FOUND
Content-Type: text/html
Content-Length: 233
Server: Werkzeug/0.11.2 Python/2.7.15rc1
Date: Mon, 30 Jul 2018 15:34:38 GMT
```

Why does this happen? Since ours is a Flask app, we can see [`app.py`]() for answers. In Flask, routes are defined with @app.route syntax. In the file, you'll see that we only have three routes defined - `/`, `/debug` and `/search`. The `/` route renders the main app, the `debug` route is used to return some debug information and finally `search` is used by the app to query elasticsearch.

```bash
$ curl 0.0.0.0:5000/debug
{
  "msg": "yellow open sfdata Ibkx7WYjSt-g8NZXOEtTMg 5 1 618 0 1.3mb 1.3mb\n",
  "status": "success"
}
```

Given that context, how would we add a new route for `hello`? You guessed it! Let's open `flask-app/app.py` in our favorite editor and make the following change

```python
@app.route('/')
def index():
  return render_template("index.html")

# add a new hello route
@app.route('/hello')
def hello():
  return "hello world!"
```

Now let's try making a request again

```bash
$ curl -I 0.0.0.0:5000/hello
HTTP/1.0 404 NOT FOUND
Content-Type: text/html
Content-Length: 233
Server: Werkzeug/0.11.2 Python/2.7.15rc1
Date: Mon, 30 Jul 2018 15:34:38 GMT
```

Oh no! That didn't work! What did we do wrong? While we did make the change in `app.py`, the file resides in our machine (or the host machine), but since Docker is running our containers based off the `prakhar1989/foodtrucks-web` image, it doesn't know about this change. To validate this, lets try the following - 

```
$ docker-compose run web bash
Starting es ... done
root@581e351c82b0:/opt/flask-app# ls
app.py        package-lock.json  requirements.txt  templates
node_modules  package.json       static            webpack.config.js
root@581e351c82b0:/opt/flask-app# grep hello app.py
root@581e351c82b0:/opt/flask-app# exit
```

What we're trying to do here is to validate that our changes are not in the `app.py` that's running in the container. We do this by running the command `docker compose run`, which is similar to its cousin `docker run` but takes additional arguments for the service (which is `web` in our [case](https://github.com/prakhar1989/FoodTrucks/blob/master/docker-compose.yml#L12)). As soon as we run `bash`, the shell opens in `/opt/flask-app` as specified in our [Dockerfile](https://github.com/prakhar1989/FoodTrucks/blob/master/Dockerfile#L13). From the grep command we can see that our changes are not in the file.

Lets see how we can fix it. First off, we need to tell docker compose to not use the image and instead use the files locally. We'll also set debug mode to `true` so that Flask knows to reload the server when `app.py` changes. Replace the `web` portion of the `docker-compose.yml` file like so:

```yaml
version: "3"
services:
  es:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.3.2
    container_name: es
    environment:
      - discovery.type=single-node
    ports:
      - 9200:9200
    volumes:
      - esdata1:/usr/share/elasticsearch/data
  web:
    build: . # replaced image with build
    command: python app.py
    environment:
      - DEBUG=True  # set an env var for flask
    depends_on:
      - es
    ports:
      - "5000:5000"
    volumes:
      - .:/opt/flask-app
volumes:
    esdata1:
      driver: local
```

With that change ([diff](https://github.com/prakhar1989/FoodTrucks/commit/31368936de5959efaf4457a94c678d21e3eefbce)), let's stop and start the containers.

```bash
$ docker-compose down -v
Stopping foodtrucks_web_1 ... done
Stopping es               ... done
Removing foodtrucks_web_1 ... done
Removing es               ... done
Removing network foodtrucks_default
Removing volume foodtrucks_esdata1

$ docker-compose up -d
Creating network "foodtrucks_default" with the default driver
Creating volume "foodtrucks_esdata1" with local driver
Creating es ... done
Creating foodtrucks_web_1 ... done
```

As a final step, lets make the change in `app.py` by adding a new route. Now we try to curl

```bash
$ curl 0.0.0.0:5000/hello
hello world
```

*** Make sure you don't have the -I argument here or you will not see the message, just headers.

Wohoo! We get a valid response! Try playing around by making more changes in the app. 

That concludes our tour of Docker Compose. With Docker Compose, you can also pause your services, run a one-off command on a container and even scale the number of containers. I also recommend you checkout a few other [use-cases](https://docs.docker.com/compose/overview/#common-use-cases) of Docker compose. Hopefully I was able to show you how easy it is to manage multi-container environments with Compose. In the final section, we are going to deploy our app to AWS!