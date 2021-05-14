Launching Solr in Solr Cloud mode

bin\solr.cmd start -e cloud

Indexing data

```
java -jar -Dc=test -Durl=http://localhost:8888/solr/update example\exampledocs\post.jar example\exampledocs\*
```

Searching

phrase "a+b"/"a%2Bb"

schema

managed schema and field guessing

creating field

```
curl -X POST -H 'Content-type:application/json' --data-binary '{"add-field": {"name":"name", "type":"text_general", "multiValued":false, "stored":true}}' http://localhost:8983/solr/films/schema
```

or use admin UI

Creating copy field for searching without specifying field

```
curl -X POST -H 'Content-type:application/json' --data-binary '{"add-copy-field" : {"source":"*","dest":"_text_"}}' http://localhost:8983/solr/films/schema
```

or use admin UI

faceting

Faceting allows the search results to be arranged into subsets (or buckets, or categories), providing a count for each subset. There are several types of faceting: field values, numeric and date ranges, pivots (decision tree), and arbitrary query faceting

range faceting

date and price

```
curl 'http://localhost:8983/solr/films/select?q=*:*&rows=0'\
    '&facet=true'\
    '&facet.range=initial_release_date'\
    '&facet.range.start=NOW-20YEAR'\
    '&facet.range.end=NOW'\
    '&facet.range.gap=%2B1YEAR'
```

currently not supported in the admin page 

Another faceting type is pivot facets, also known as "decision trees", allowing two or more fields to be nested for all the various possible combinations. 

```
bin/solr delete -c films
```

indexing your own data

creating your own collection

```
./bin/solr create -c localDocs -s 2 -rf 2
./bin/post -c localDocs ~/Documents
```

 [Data Import Handler (DIH)](https://solr.apache.org/guide/8_8/uploading-structured-data-store-data-with-the-data-import-handler.html#uploading-structured-data-store-data-with-the-data-import-handler) to connect to databases, mail servers, ...

even if you index content in this tutorial more than once, it does not duplicate the results found. This is because the example Solr schema (a file named either `managed-schema` or `schema.xml`) specifies a `uniqueKey` field called `id`.

Whenever you POST commands to Solr to add a document with the same value for the `uniqueKey` as an existing document, it automatically replaces it for you.

`numDocs` and `maxDoc`

`numDocs` represents the number of searchable documents in the index

The `maxDoc` value may be larger as the `maxDoc` count includes logically deleted documents that have not yet been physically removed from the index. 

deleting data

```
bin/post -c localDocs -d "<delete><id>SP2514N</id></delete>"
bin/post -c localDocs -d "<delete><query>*:*</query></delete>"
```

spatial query

Solr has sophisticated geospatial support, including searching within a specified distance range of a given location (or within a bounding box), sorting by distance, or even boosting results by the distance.

#### Solr control script reference

Solr includes a script known as “bin/solr” that allows you to perform many common operations on your Solr installation or cluster.

You can start and stop Solr, create and delete collections or cores, perform operations on ZooKeeper and check the status of Solr and configured shards.

##### Starting and restart

The `start` command starts Solr. The `restart` command allows you to restart Solr while it is already running or if it has been stopped already.

```
bin/solr start [options]
bin/solr start -help
bin/solr restart [options]
bin/solr restart -help
```

When using the `restart` command, you must pass all of the parameters you initially passed when you started Solr.

-a/-D "" additional parameters for JVM

-c Start Solr in SolrCloud mode, which will also launch the embedded ZooKeeper instance included with Solr.

-d <dir>  Define a server directory, defaults to `server` (as in, `$SOLR_HOME/server`). (more common to use the same server directory for each instance and use a unique Solr home directory using the -s option)

-e <name> example configuration

-h <hostname> Start Solr with the defined hostname. If this is not specified, 'localhost' will be assumed.

```
bin/solr start -h search.mysolr.com
```

-m <memory>

-p <port> default:8983

-s <dir> Sets the `solr.solr.home` system property If set, the specified directory should contain a `solr.xml` file, unless `solr.xml` exists in ZooKeeper. The default value is `server/solr`.

-v bin/solr start -f -v INFO->Debug

-q Info->Warning

-z <zkhost> bin/solr start -c -z server1:2181,server2:2181

##### Stop

-p <port>

-all

-key <key>

##### Version

##### Status

The `status` command displays basic JSON-formatted information for any Solr nodes found running on the local system.

##### Assert

The `assert` command sanity checks common issues with Solr installations.

##### Healthcheck

The `healthcheck` command generates a JSON-formatted health report for a collection when running in SolrCloud mode

-c collection

-z zookeeper host

##### Collections and cores

The `create` command detects the mode that Solr is running in (standalone or SolrCloud) and then creates a core or collection depending on the mode.

-c name

-d confdir 

-n configname This defaults to the same name as the core or collection.

-p port

-s shards

-rf <replicas> or -replicationFactor

##### Configuration Directories and SolrCloud

Before creating a collection in SolrCloud, the configuration directory used by the collection must be uploaded to ZooKeeper.

delete core or collection

##### Authentication

```
bin/solr auth enable
```

```
-credentials
```

The username and password in the format of `username:password` of the initial user.

```
-blockUnknown
```

When **true**, blocks all unauthenticated users from accessing Solr. This defaults to **false**, which means unauthenticated users will still be able to access Solr.

##### Set or Unset Configuration Properties

-name -action -property -value -z(ookeeper) -p(ort)

#### Solr configuration files

Solr has several configuration files that you will interact with during your implementation.

Many of these files are in XML format, although APIs that interact with configuration settings tend to accept JSON for programmatic access as needed.

*Standalone Mode crucial files*

```
<solr-home-directory>/
   solr.xml
   core_name1/
      core.properties
      conf/
         solrconfig.xml
         managed-schema
      data/
   core_name2/
      core.properties
      conf/
         solrconfig.xml
         managed-schema
      data/
```

- `solr.xml` specifies configuration options for your Solr server instance. For more information on `solr.xml` see [Solr Cores and solr.xml](https://solr.apache.org/guide/8_8/solr-cores-and-solr-xml.html#solr-cores-and-solr-xml).
- Per Solr Core:
  - `core.properties` defines specific properties for each core such as its name, the collection the core belongs to, the location of the schema, and other parameters. For more details on `core.properties`, see the section [Defining core.properties](https://solr.apache.org/guide/8_8/defining-core-properties.html#defining-core-properties).
  - `solrconfig.xml` controls high-level behavior. You can, for example, specify an alternate location for the data directory. For more information on `solrconfig.xml`, see [Configuring solrconfig.xml](https://solr.apache.org/guide/8_8/configuring-solrconfig-xml.html#configuring-solrconfig-xml).
  - `managed-schema` (or `schema.xml` instead) describes the documents you will ask Solr to index. The Schema define a document as a collection of fields. You get to define both the field types and the fields themselves. Field type definitions are powerful and include information about how Solr processes incoming field values and query values. For more information on Solr Schemas, see [Documents, Fields, and Schema Design](https://solr.apache.org/guide/8_8/documents-fields-and-schema-design.html#documents-fields-and-schema-design) and the [Schema API](https://solr.apache.org/guide/8_8/schema-api.html#schema-api).
  - `data/` The directory containing the low level index files.

#### Taking Solr to Production

Solr includes a service installation script (`bin/install_solr_service.sh`) to help you install Solr as a service on Linux

We recommend separating your live Solr files, such as logs and index files, from the files included in the Solr distribution bundle, as that makes it easier to upgrade Solr and is considered a good practice to follow as a system administrator.

separate writable files? -d to change directory

-u to set username

```bash
tar xzf solr-8.8.0.tgz solr-8.8.0/bin/install_solr_service.sh --strip-components=2
```

#### Documents fields and schemas

Solr’s basic unit of information is a *document*, which is a set of data that describes something

documents are composed of *fields*, which are more specific pieces of information

You can tell Solr about the kind of data a field contains by specifying its *field type*. The field type tells Solr how to interpret the field and how it can be queried.

*Field analysis* tells Solr what to do with incoming data when building an index.

Solr stores details about the field types and fields it is expected to understand in a schema file

`managed-schema` is the name for the schema file Solr uses by default to support making Schema changes at runtime via the [Schema API](https://solr.apache.org/guide/8_8/schema-api.html#schema-api), or [Schemaless Mode](https://solr.apache.org/guide/8_8/schemaless-mode.html#schemaless-mode) features. the contents of the files are still updated automatically by Solr.

`schema.xml` is the traditional name for a schema file which can be edited manually by users who use the [`ClassicIndexSchemaFactory`](https://solr.apache.org/guide/8_8/schema-factory-definition-in-solrconfig.html#schema-factory-definition-in-solrconfig)

##### Solr field types

A field type defines the analysis that will occur on a field when documents are indexed or queries are sent to the index.

A field type definition can include four types of information:

- The name of the field type (mandatory).
- An implementation class name (mandatory).
- If the field type is `TextField`, a description of the field analysis for the field type.
- Field type properties - depending on the implementation class, some properties may be mandatory

Field types are defined in `schema.xml`. Each field type is defined between `fieldType` elements. They can optionally be grouped within a `types` element. Here is an example of a field type definition for a type called `text_general`

```
<fieldType name="text_general" class="solr.TextField" positionIncrementGap="100"> 
  <analyzer type="index"> 
```

The rest of the definition is about field analysis

The field type `class` determines most of the behavior of a field type, but optional properties can also be defined. For example, the following definition of a date field type defines two properties, `sortMissingLast` and `omitNorms`.

general properties: name, class, ...

##### Fields

Fields are defined in the fields element of `schema.xml`. Once you have the field types set up, defining the fields themselves is simple.

```
<field name="price" type="float" default="0.0" indexed="true" stored="true"/>
```

##### Copying fields

You might want to interpret some document fields in more than one way. Solr has a mechanism for making copies of fields so that you can apply several distinct field types to a single piece of incoming information.

```
<copyField source="cat" dest="text" maxChars="30000" />
```

A common usage for this functionality is to create a single "search" field that will serve as the default query field when users or clients do not specify a field to query. For example, `title`, `author`, `keywords`, and `body` may all be fields that should be searched by default, with copy field rules for each field to copy to a `catchall` field

Later you can set a rule in `solrconfig.xml` to search the `catchall` field by default. 

The `maxChars` parameter, an `int` parameter, establishes an upper limit for the number of characters to be copied from the source value when constructing the value added to the destination field. This limit is useful for situations in which you want to copy some data from the source field, but also control the size of index files.

##### Dynamic fields

*Dynamic fields* allow Solr to index fields that you did not explicitly define in your schema.

##### Unique keys

The `uniqueKey` element specifies which field is a unique identifier for documents.

```
<uniqueKey>id</uniqueKey>
```





##### Overall structure of schema.xml

At the highest level, `schema.xml` is structured as follows.

This example is not real XML, but it gives you an idea of the structure of the file.

```xml
<schema>
  <types>
  <fields>
  <uniqueKey>
  <copyField>
</schema>
```





#### Flask

 The “micro” in microframework means Flask aims to keep the core simple but extensible.

```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, World!'
```

import Flask class, \_\_name\_\_ to decide whether it's started as an application or imported as module.

route() decorator to tell Flask what URL should trigger the function

```
$ export FLASK_APP=hello.py
$ flask run
 * Running on http://127.0.0.1:5000/
 
 set FLASK_APP=hello.py #windows
 $env:FLASK_APP = "hello.py" #powershell
```

##### routing

```python
@app.route('/hello')
def hello():
    return 'Hello, World'
```

Use the route() decorator to bind a function to a URL.

You can add variable sections to a URL by marking sections with `<variable_name>`. Your function then receives the `<variable_name>` as a keyword argument. Optionally, you can use a converter to specify the type of the argument like `<converter:variable_name>`.

```python
@app.route('/post/<int:post_id>')
def show_post(post_id):
    # show the post with the given id, the id is an integer
    return 'Post %d' % post_id

@app.route('/path/<path:subpath>')
def show_subpath(subpath):
    # show the subpath after /path/
    return 'Subpath %s' % escape(subpath)
```

Converter types

| `string` | (default) accepts any text without a slash |
| -------- | ------------------------------------------ |
| `int`    | accepts positive integers                  |
| `float`  | accepts positive floating point values     |
| `path`   | like `string` but also accepts slashes     |
| `uuid`   | accepts UUID strings                       |

##### Unique URLs/ Redirection 

If you access the URL without a trailing slash, Flask redirects you to the canonical URL with the trailing slash.

##### URL Building

To build a URL to a specific function, use the [`url_for()`](https://flask.palletsprojects.com/en/1.1.x/api/#flask.url_for) function. It accepts the name of the function as its first argument and any number of keyword arguments, each corresponding to a variable part of the URL rule. Unknown variable parts are appended to the URL as query parameters.

1. You can change your URLs in one go instead of needing to remember to manually change hard-coded URLs.
2. URL building handles escaping of special characters and Unicode data transparently.
3. The generated paths are always absolute, avoiding unexpected behavior of relative paths in browsers.

```python
from markupsafe import escape
@app.route('/user/<username>')
def profile(username):
    return '{}\'s profile'.format(escape(username))
```

##### HTTP methods

You can use the `methods` argument of the route() decorator to  handle different HTTP methods.

```python
from flask import request

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        return do_the_login()
    else:
        return show_the_login_form()
```

##### Static files

 create a folder called `static` in your package or next to your module and it will be available at `/static` on the application.

```
url_for('static', filename='style.css') #to generate URLs for static files
```

To render a template you can use the render_template()  # from jinja2

```python
from flask import render_template

@app.route('/hello/')
@app.route('/hello/<name>')
def hello(name=None):
    return render_template('hello.html', name=name)
```

Flask will look for templates in the `templates` folder.

```html
<!doctype html>
<title>Hello from Flask</title>
{% if name %}
  <h1>Hello {{ name }}!</h1>
{% else %}
  <h1>Hello, World!</h1>
{% endif %}
```

Inside templates you also have access to the request, session and g 1 objects as well as the get_flashed_messages() function.

##### Accessing request data with context locals

```python
from flask import request
@app.route('/login', methods=['POST', 'GET'])
def login():
    error = None
    if request.method == 'POST':
        if valid_login(request.form['username'],
                       request.form['password']):
            return log_the_user_in(request.form['username'])
        else:
            error = 'Invalid username/password'
    # the code below is executed if the request method
    # was GET or the credentials were invalid
    return render_template('login.html', error=error)
```

To access parameters submitted in the URL (`?key=value`) you can use the [`args`](https://flask.palletsprojects.com/en/1.1.x/api/#flask.Request.args) attribute:

```
searchword = request.args.get('key', '')
```

Each uploaded file is stored in that dictionary. It behaves just like a standard Python `file` object, but it also has a [`save()`](https://werkzeug.palletsprojects.com/en/1.0.x/datastructures/#werkzeug.datastructures.FileStorage.save)hat allows you to store that file on the filesystem of the server. 

```python
from flask import request
from werkzeug.utils import secure_filename
@app.route('/upload', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        f = request.files['the_file']
        f.save('/var/www/uploads/uploaded_file.txt')
        f.save('/var/www/uploads/' + secure_filename(f.filename))
```

##### Cookies

```python
from flask import make_response

@app.route('/xxx')
def index():
    resp = make_response(render_template(...))
    resp.set_cookie('username', 'the username')
    return resp
    
@app.route('/')
def index():
    username = request.cookies.get('username')
    # use cookies.get(key) instead of cookies[key] to not get a
    # KeyError if the cookie is missing.
```

##### Redirect

```python
from flask import abort, redirect, url_for

@app.route('/')
def index():
    return redirect(url_for('login'))

@app.route('/login')
def login():
    abort(401)
    this_is_never_executed()
    
@app.errorhandler(404)
def page_not_found(error):
    return render_template('page_not_found.html'), 404
```

The return value from a view function is automatically converted into a response object for you. If the return value is a string it’s converted into a response object with the string as response body, a `200 OK` status code and a *text/html* mimetype. If the return value is a dict, `jsonify()` is called to produce a response. 

jsonify to convert it explicitly

#### Postman









#### Conda

```
conda create --name myenv python=3.6 scipy=0.15.0
conda create --name myclone --clone myenv
conda create --name myenv --file spec-file.txt
conda install --name myenv --file spec-file.txt
conda env export > environment.yml
conda env create -f environment.yml
conda env list
conda list --explicit
conda list --explicit > spec-file.txt
conda activate ...
conda deactivate ...
conda list --revisions
conda install --revision=REVNUM or conda install --rev REVNUM
conda remove --name myenv --all
```



#### Docker

Simply put, a container is simply another process on your machine that has been isolated from all other processes on the host machine. That isolation leverages [kernel namespaces and cgroups](https://medium.com/@saschagrunert/demystifying-containers-part-i-kernel-space-2c53d6979504), features that have been in Linux for a long time. Docker has worked to make these capabilities approachable and easy to use.

When running a container, it uses an isolated filesystem. This custom filesystem is provided by a **container image**. Since the image contains the container’s filesystem, it must contain everything needed to run an application - all dependencies, configuration, scripts, binaries, etc. The image also contains other configuration for the container, such as environment variables, a default command to run, and other metadata.

##### Building an APP with docker

first, download the app contents

next build the app's container image

```
 # syntax=docker/dockerfile:1
 FROM node:12-alpine
 RUN apk add --no-cache python g++ make
 WORKDIR /app
 COPY . .
 RUN yarn install --production
 CMD ["node", "src/index.js"]
```

then build the container image

```
docker build -t getting-started .
docker image ls
```

finally start the container

```
 docker run -dp 3000:3000 getting-started
```

-d for detached mode, -p for port mapping

##### Updating the APP

We need to remove the old container first before adding a new one

```
docker ps #to view the container list
docker stop <container-id> #to stop a container
docker remove <contrainer-id> #to erase a container or use docker rm -f <container-id>
```

then build the new image and start the new app

or?

##### Docker Hub to share application

docker hub is the default registry for docker

docker id required

to push an image to docker hub

```
docker login -u Your-Username
docker tag getting-started YOUR-USER-NAME/getting-started #add tag
docker push YOUR-USER-NAME/getting-started
```

play-with-docker: a sandbox platform for docker

##### Persisting the database

containers have isolated file systems

```
docker run -d ubuntu bash -c "shuf -i 1-10000 -n 1 -o /data.txt && tail -f /dev/null"
docker exec <container-id> cat /data.txt
```

container volumes can provide the ability to connect specific filesystem paths of the container back to the host machine.

named volume and bind mount and more

create a volume 

```
 docker volume create todo-db
```

start the container with -v flag

```
 docker run -dp 3000:3000 -v todo-db:/etc/todos getting-started
```

to check volume storage location on the host

```
docker volume inspect todo-db
```

with bind mounts, we control the exact mountpoint on host. It can also be used to provide additional data into containers

- Mount our source code into the container
- Install all dependencies, including the “dev” dependencies
- Start nodemon to watch for filesystem changes

```
 docker run -dp 3000:3000 \
     -w /app -v "$(pwd):/app" \
     node:12-alpine \
     sh -c "yarn install && yarn run dev"
```

##### Multi container apps

each container in general should do one thing, run one process

If two containers are on the same network, they can talk to each other

There are two ways to put a container on a network: 1) Assign it at start or 2) connect an existing container

create the network

```
 docker network create todo-app
```

start MySQL container and attach it to the network

```
 docker run -d \
     --network todo-app --network-alias mysql \
     -v todo-mysql-data:/var/lib/mysql \
     -e MYSQL_ROOT_PASSWORD=secret \
     -e MYSQL_DATABASE=todos \
     mysql:5.7
```

--network-alias?

To connect to the MySQL container, we’re going to make use of the `nicolaka/netshoot` container, which ships with a *lot* of tools that are useful for troubleshooting or debugging networking issues

```
docker run -it --network todo-app nicolaka/netshoot
```

Inside the container, we’re going to use the `dig` command, which is a useful DNS tool. We’re going to look up the IP address for the hostname `mysql`

```
 dig mysql
```

connect the app with mysql

```
 docker run -dp 3000:3000 \
   -w /app -v "$(pwd):/app" \
   --network todo-app \
   -e MYSQL_HOST=mysql \
   -e MYSQL_USER=root \
   -e MYSQL_PASSWORD=secret \
   -e MYSQL_DB=todos \
   node:12-alpine \
   sh -c "yarn install && yarn run dev"
   
docker logs <container-id>
```

##### Docker compose

[Docker Compose](https://docs.docker.com/compose/) is a tool that was developed to help define and share multi-container applications.

define version and services(or containers) in docker-compose.yml

```
 version: "3.7"

 services:
   app:
     image: node:12-alpine
     command: sh -c "yarn install && yarn run dev"
     ports:
        - 3000:3000
     working_dir: /app
     volumes:
       - ./:/app
     environment:
       MYSQL_HOST: mysql
       MYSQL_USER: root
       MYSQL_PASSWORD: secret
       MYSQL_DB: todos
   mysql:
       image:mysql:5.7
       volumes:
         - todo-mysql-data:/var/lib/mysql
       environment:
          MYSQL_ROOT_PASSWORD:secret
          MYSQL_DATABASE:todos
   volumes:
       todo-mysql-data
```

the service name will automatically become a network alias

commands, ports, working_dir and volumes and environment variables can also be defined in the file

then run the application stack

```
docker-compose up -d
docker-compose logs -f
```

-d means run everything in the background

a network will be automatically created

```
docker-compose down
```

to tear it down, --volumes to also remove volumes

##### Image building Best practices

When you have built an image, it is good practice to scan it for security vulnerabilities using the `docker scan` command.

Use the `docker image history` command to see the layers in the `getting-started` image you created earlier in the tutorial.

```
 docker image history --no-trunc getting-started
```

we need to restructure our Dockerfile to help support the caching of the dependencies. For Node-based applications, those dependencies are defined in the `package.json` file.

#### Docker building python projects

shell and exec forms

RUN, CMD and ENTRYPOINT can be specified in shell form or exec form.

shell form:

```
<instruction>   <command>
RUN apt-get install python3
```

Exec form:

```
<instruction> ["executable","param1"."param2"]
RUN ["apt-get","install","python3"]
```

in exec form shell processing doesn't happen

RUN instruction allows you to install your application and packages requited for it. It executes any commands on top of the current image and creates a new layer by committing the results.

CMD instruction allows you to set a *default* command, which will be executed only when you run container without specifying a command. If Docker container runs with a command, the default command will be ignored. If Dockerfile has more than one CMD instruction, all but last CMD instructions are ignored.

ENTRYPOINT instruction allows you to configure a container that will run as an executable.

It looks similar to CMD, because it also allows you to specify a command with parameters. The difference is ENTRYPOINT command and parameters are not ignored when Docker container runs with command line parameters.





##### Building an image

Dockerfile in the root of the working directory for instructions in building images

```
# syntax=docker/dockerfile:1
```

instructs the Docker builder what syntax to use when parsing the Dockerfile

```
FROM python:3.8-slim-buster
```

defines the base image to use in the application (official Python image that already has all the tools and packages that we need to run a Python application)

```
WORKDIR /app
```

defines the working directory for Docker to use as the default location for all subsequent commands

Usually, the very first thing you do once you’ve downloaded a project written in Python is to install `pip` packages.

Before we can run `pip3 install`, we need to get our `requirements.txt` file into our image. We’ll use the `COPY` command to do this. The `COPY` command takes two parameters. The first parameter tells Docker what file(s) you would like to copy into the image. The second parameter tells Docker where you want that file(s) to be copied to. We’ll copy the `requirements.txt` file into our working directory `/app`.

```
COPY requirements.txt requirements.txt
```

```
RUN pip3 install -r requirements.txt
```

to install dependencies

```
COPY . .
```

copy all the files (including source code) located in the current directory into the image

```
CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0"]
```

run when the image is executed inside a container

--host=0.0.0.0 to make the application externally visible

```
python-docker
|____ app.py
|____ requirements.txt
|____ Dockerfile
```

↑Directory structure

building the image

```
docker build --tag python-docker .
```

The `docker build` command builds Docker images from a Dockerfile and a “context”. A build’s context is the set of files located in the specified PATH or URL. The Docker build process can access any of the files located in this context.

--tag to set the name of the image, name:tag format, latest as the default tag

```
docker images
```

to list all images

```
$ docker tag python-docker:latest python-docker:v1.0.0
```

to create a second tag for the image

```
docker rmi python-docker:v1.0.0
```

removing an image

##### Running containers

A container is a normal operating system process except that this process is isolated in that it has its own file system, its own networking, and its own isolated process tree separate from the host.

To run an image inside of a container, we use the `docker run` command.

```
$ docker run python-docker
```

To publish a port for our container, we’ll use the `--publish flag` (`-p` for short) on the `docker run` command.

The format of the `--publish` command is `[host port]:[container port]`.

running in detached mode

Docker can run your container in detached mode or in the background

```
docker run -d -p 5000:5000 --name rest-server python-docker
```

```
docker ps -a
```

to list containers

containers can be stopped, started and restarted, --all/-a for all containers, rm to remove containers

Docker generated a random name if you don't name the container.

--name to add a name

##### Develop an app

```
$ docker volume create mysql
$ docker volume create mysql_config
```

volumes to persist data

create a network that our application and database will use to talk to each other. The network is called a user-defined bridge network and gives us a nice DNS lookup service which we can use when creating our connection string.

```
$ docker network create mysqlnet
```

```
$ docker run --rm -d -v mysql:/var/lib/mysql \
  -v mysql_config:/etc/mysql -p 3306:3306 \
  --network mysqlnet \
  --name mysqldb \
  -e MYSQL_ROOT_PASSWORD=p@ssw0rd1 \
  mysql
```

```
docker exec -it mysqldb mysql -u root -p
```

```python
@app.route('/widgets')
def get_widgets() :
  mydb = mysql.connector.connect(
    host="mysqldb",
    user="root",
    password="p@ssw0rd1",
    database="inventory"
  )
  cursor = mydb.cursor()


  cursor.execute("SELECT * FROM widgets")

  row_headers=[x[0] for x in cursor.description] #this will extract row headers

  results = cursor.fetchall()
  json_data=[]
  for result in results:
    json_data.append(dict(zip(row_headers,result)))

  cursor.close()

  return json.dumps(json_data)

@app.route('/initdb')
def db_init():
  mydb = mysql.connector.connect(
    host="mysqldb",
    user="root",
    password="p@ssw0rd1"
  )
  cursor = mydb.cursor()

  cursor.execute("DROP DATABASE IF EXISTS inventory")
  cursor.execute("CREATE DATABASE inventory")
  cursor.close()

  mydb = mysql.connector.connect(
    host="mysqldb",
    user="root",
    password="p@ssw0rd1",
    database="inventory"
  )
  cursor = mydb.cursor()

  cursor.execute("DROP TABLE IF EXISTS widgets")
  cursor.execute("CREATE TABLE widgets (name VARCHAR(255), description VARCHAR(255))")
  cursor.close()

  return 'init database'
```

```shell
$ docker run \
  --rm -d \
  --network mysqlnet \
  --name rest-server \
  -p 5000:5000 \
  python-docker
  
$ curl http://localhost:5000/initdb
$ curl http://localhost:5000/widgets
```

```
version: '3.8'

services:
 web:
  build:
   context: .
  ports:
  - 5000:5000
  volumes:
  - ./:/app

 mysqldb:
  image: mysql
  ports:
  - 3306:3306
  environment:
  - MYSQL_ROOT_PASSWORD=p@ssw0rd1
  volumes:
  - mysql:/var/lib/mysql
  - mysql_config:/etc/mysql

volumes:
  mysql:
  mysql_config:
```

##### Configuring CI/CD

add DOCKER_HUB_USERNAME and DOCKER_HUB_ACCESS_TOKEN into GitHub secrets UI to ensure we can access Docker Hub from any workflow

set up the GitHub actions workflow

We need two Docker actions:

1. The first action enables us to log in to Docker Hub using the secrets we stored in the GitHub Repository.
2. The second one is the build and push action.

To set up the workflow:

1. Go to your repository in GitHub and then click **Actions** > **New workflow**.
2. Click **set up a workflow yourself** and add the following content: name, branch, jubs, steps

```
name: CI to Docker Hub


on:
  push:
    branches: [ main ]
    
jobs:

  build:
    runs-on: ubuntu-latest
    
 steps:

      - name: Check Out Repo 
        uses: actions/checkout@v2

      - name: Login to Docker Hub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKER_HUB_USERNAME }}
          password: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}

      - name: Set up Docker Buildx
        id: buildx
        uses: docker/setup-buildx-action@v1

      - name: Build and push
        id: docker_build
        uses: docker/build-push-action@v2
        with:
          context: ./
          file: ./Dockerfile
          push: true
          tags: ${{ secrets.DOCKER_HUB_USERNAME }}/simplewhale:latest

      - name: Image digest
        run: echo ${{ steps.docker_build.outputs.digest }}
    
```



### React Router

Features like browser history, bookmarks, and forward and back buttons will not work without a routing solution.

sitemap to define the structure of web pages

React Router DOM is used for regular React applications that use the DOM. If you’re writing an app for React Native, you’ll use `react-router-native`.

The `Router` component passes information about the current location to any children that are nested inside of it. 

```react 
import { BrowserRouter as Router } from "react-router-dom";
<Router>
    <App />
</Router>
```

configuring routes

```react
import { Routes, Route } from "react-router-dom";
return(    
    <div>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route
          path="/about"
          element={<About />}
        />
        <Route
          path="/events"
          element={<Events />}
        />
        <Route
          path="/products"
          element={<Products />}
        />
        <Route
          path="/contact"
          element={<Contact />}
        />
        <Route path="*" element={<Whoops404 />} />
      </Routes>
    </div>
);
```

Each `Route` component has `path` and `element` properties. When the browser’s location matches the `path`, the `element` will be displayed. 

creating links

```react
import { Link } from "react-router-dom";

export function Home() {
  return (
    <div>
      <h1>[Company Website]</h1>
      <nav>
        <Link to="about">About</Link>
        <Link to="events">Events</Link>
        <Link to="products">Products</Link>
        <Link to="contact">Contact Us</Link>
      </nav>
    </div>
  );
}
```

getting current location from hooks

```react
let location = useLocation();
{location.pathname}
```

creating a page hierarchy

```react
<Route path="about" element={<About />}>
     <Route
        path="services"
        element={<Services />}
     />

     <Route
        path="history"
        element={<History />}
     />
     <Route
        path="location"
        element={<Location />}
     />
</Route>
```

In order to get component `history` to display, we’ll use another feature of React Router DOM: the `Outlet` component.

`Outlet` will let us render these nested components. We’ll just place it anywhere we want to render child content.

```react
import {
  Link,
  useLocation,
  Outlet
} from "react-router-dom";

export function About() {
  return (
    <div>
      <h1>[About]</h1>
      <Outlet />
    </div>
  );
}
```

redirect

```react
<Routes>
        <Route path="/" element={<Home />} />
        // Other Routes
        <Redirect
          from="services"
          to="about/services"
        />
</Routes>
```



useRoute to create routes with hooks

```react
import { useRoutes } from "react-router-dom";
let element = useRoutes([
    { path: "/", element: <Home /> },
    ...
]);
return element;
```



Routing parameters

Another useful feature of the React Router is the ability to set up *routing parameters*. Routing parameters are variables that obtain their values from the URL.

```react
<Routes>
        <Route
          path="/"
          element={<ColorList />}
        />
        <Route
          path=":id"
          element={<ColorDetails />}
        />
</Routes>

import React from "react";
import { useParams } from "react-router-dom";

export function ColorDetails() {
  let params = useParams();
    let { id } = useParams();
  let { colors } = useColors();
  let foundColor = colors.find(
    color => color.id === id
  );
  console.log(params);
  return (
    <div>
      <h1>Details</h1>
    </div>
  );
}
```

hook to navigate

```react
let navigate = useNavigate();

return (
  <section
    className="color"
    onClick={() => navigate(`/${id}`)}
  >
    // Color component
  </section>
);
```

### React and server

React can be rendered *isomorphically*, which means that it can be in platforms other than the browser. This means we can render our UI on the server before it ever gets to the browser.

*Isomorphic* applications are applications that can be rendered on multiple platforms. *Universal* code means that the exact same code can run in multiple environments.

Node.js will allow us to reuse the same code we’ve written in the browser in other applications such as servers, CLIs, and even native applications.

```react
let html = ReactDOM.renderToString(<Star />);
```

### Java

major advantage: write once and run on all devices

better suited to development in the large

new features: modules and interactive JShell

##### Setting up Java environment

The JRE (Java Runtime Environment) was, up until Java 8, a smaller download for end users. Since there is far less desktop Java than there once was, the JRE was eliminated in favor of `jlink` to make a custom download.

The JDK or Java SDK download is the full development environment, which you’ll want if you’re going to be developing Java software.

Using the command-line Java Development Kit (JDK) may be the best way to keep up with the very latest improvements in Java.

```
javac HelloWorld.java

java HelloWorld
```

There is an optional setting called `CLASSPATH`, discussed in [Recipe 1.5](https://learning.oreilly.com/library/view/java-cookbook-4th/9781492072577/ch01.html#javacook-getstarted-SECT-4), that controls where Java looks for classes. 

`CLASSPATH`, if set, is used by both *javac* and *java*.

GraalVM

better performance, mix and match programming languages

or use IDEs

##### JShell

REPL interpreter of Java starting with Java 11

tab for auto completion, double tab to see documentation

##### CLASSPATH

`CLASSPATH` is a list of class files in any of a number of directories, JAR files, or ZIP files.

Java platform modules system JPMS: major feature in Java 9

##### JVM

JVM starts an interpreter for bytecode from the program, while production JVMs also provide a runtime compiler that will accelerate the important parts of the program by replacing them with equivalent compiled machine code.

- Comprise a container for application code to run inside
- Provide a secure and reliable execution environment as compared to C/C++
- Take memory management out of the hands of developers
- Provide a cross-platform execution environment
- make use of runtime information to self-manage

JVM collects runtime information to make better decisions about how to execute code.

just-in-time (JIT) compilation

In the HotSpot JVM (which was the JVM that Sun first shipped as part of Java 1.3, and is still in use today), the JVM first identifies which parts of the program are called most often—the “hot methods.” Then, the JVM compiles these hot methods directly into machine code, bypassing the JVM interpreter.

##### Java security

the architecture itself is strong and robust, without any security holes in the design(at least none that have been discovered yet).

Fundamental to the design of the security model is that bytecode is heavily restricted in what it can express—there is no way, for example, to directly address memory.

Furthermore, the VM goes through a process known as *bytecode* *verification* whenever it loads an untrusted class, which removes a further large class of problems

For practical server-side coding, Java remains perhaps the most secure general-purpose platform currently available, especially when kept patched up to date.

java is statically typed

java is always pass-by-value (except objects)

java is multithreaded compared to python

