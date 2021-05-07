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

