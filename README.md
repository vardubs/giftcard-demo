# giftcard-demo-series
The Axon Framework Giftcard demo applications focus around a simple gift card domain, designed to show various aspects of the framework. The app can be run in various modes, using [Spring-boot Profiles](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-profiles.html): by selecting a specific profile, only the corresponding parts of the app will be active. Select none, and the default behaviour is activated, which activates everything. This way you can experiment with Axon in a (structured) monolith as well as in micro-services.

## The Gifcard app

### Background story
See [the wikipedia article](https://en.wikipedia.org/wiki/Gift_card) for a basic definition of gift cards. Essentially, there are just two events in the life cycle of a gift card:
* They get _issued_: a new gift card gets created with some amount of money stored.
* They get _redeemed_: all or part of the monetary value stored on the gift card is used to purchase something.

### Structure of the App
The Giftcard application is split into four parts, using four sub-packages of `io.axoniq.demo.giftcard`:
* The `api` package contains the ([Kotlin](https://kotlinlang.org/)) sourcecode of the messages and entity. They form the API (sic) of the application.
* The `command` package contains the GiftCard Aggregate class, with all command- and associated eventsourcing handlers.
* The `query` package provides the query handlers, with their associated event handlers.
* The `gui` package contains the [Vaadin](https://vaadin.com/)-based Web GUI.

Of these packages, `command`, `query`, and `gui` are also configured as profiles.

### Building the Giftcard app from the sources
To build the demo app, simply run the provided [Maven wrapper](https://www.baeldung.com/maven-wrapper):

    ```
        mvnw clean package
    ```
Note that for Mac OSX or Linux you probably have to add "`./`" in front of `mvnw`.

## Running the Giftcard app

The simplest way to run the app is by using the Spring-boot maven plugin:

    ```
        mvnw spring-boot:run
    ```
However, if you have copied the jar file `giftcard-distributed-1.0.jar` from the Maven `target` directory to some other location, you can also start it with:

    ```
        java -jar giftcard-distributed-1.0.jar
    ```
The Web GUI can be found at [`http://localhost:8080`](http://localhost:8080).

If you want to activate only the `command` profile, use:

    ```
        java -Dspring.profiles.active=command giftcard-distributed-1.0.jar
    ```
Idem for `query` and `gui`.

### Running the Giftcard app as micro-services

To run the Giftcard app as if it were three seperate micro-services, use the Spring-boot `spring.profiles.active` option as follows:

    ```
        $ java -Dspring.profiles.active=command -jar giftcard-distributed-1.0.jar
    ```
This will start only the command part. To complete the app, open two other command shells, and start one with profile `query`, and the last one with `gui`. Again you can open the Web GUI at [`http://localhost:8080`](http://localhost:8080). The three parts of the application work together through the running instance of the Axon Server, which distributes the Commands, Queries, and Events.

## Running Axon Server

By default the Axon Framework is configured to expect a running Axon Server instance, and it will complain if the server is not found. To run Axon Server, you'll need a Java runtime (JRE versions 8 through 10 are currently supported, Java 11 still has Spring-boot related growing-pains).  A copy of the server JAR file has been provided in the demo package. You can run it locally, in a Docker container (including Kubernetes or even Mini-kube), or on a separate server.

### Running Axon Server locally

To run Axon Server locally, all you need to do is put the server JAR file in the directory where you want it to live, and start it using:

    ```
        java -jar axonserver-4.0-exec.jar
    ```

You will see that it creates a subdirectory `data` where it will store its information.

### Running Axon Server in a Docker container

To run Axon Server in Docker you can use the image provided on Docker Hub:

    ```
        $ docker run -d --name my-axon-server -p 8024:8024 -p 8124:8124 axoniq/axonserver
        ...some container id...
        $
    ```

*WARNING* This is not a supported image for production purposes. Please use with caution.

This image will force Axon Server to use "`localhost`" as its own hostname by default, so applications outside the container can reach it. If you want to run the clients in Docker containers as well, and are not using something like Kubernetes, use the "`--hostname`" option of the `docker` command to set a useful name like "axonserver", and pass the `AXONSERVER_HOSTNAME` environment variable to adjust the properties accordingly:

    ```
        $ docker run -d --name my-axon-server -p 8024:8024 -p 8124:8124 --hostname axonserver -e AXONSERVER_HOSTNAME=axonserver axoniq/axonserver
    ```

When you start the client containers, you can now use "`--link axonserver`" to provide them with the correct DNS entry. The Axon Server-connector looks at the "`axon.axonserver.servers`" property to determine where Axon Server lives, so don't forget to set it to "`axonserver`".

### Running Axon Server in Kubernetes and Mini-Kube

*WARNING*: Although you can get a pretty functional cluster running locally using Mini-Kube, you can run into trouble when you want to let it serve clients outside of the cluster. Mini-Kube can provide access to HTTP servers running in the cluster, for other protocols you have to run a special protocol-agnostic proxy like you can with "`kubectl port-forward` _&lt;pod-name&gt;_ _&lt;port-number&gt;_". For non-development scenarios, we don't recommend using Mini-Kube.

You can use the same files in the "`docker`" directory also for running Axon Server in Kubernetes. Assuming you have access to a working cluster, either using Mini-Kube or with a 'real' setup, make sure you have the Docker environment variables set correctly to use it. With Mini-Kube this is done using the "`minikube docker-env`" command, with the Google Cloud SDK use "`gcloud auth configure-docker`".

Deployment requires the use of a YAML descriptor, an example of which can be found in the "`kubernetes`" directory.

## Configuring Axon Server

Axon Server uses sensible defaults for all of its settings, so it will actually run fine without any further configuration. However, if you want to make some changes, below are the most common options.

### Environment variables for customizing the Docker image of Axon Server

The `axoniq/axonserver` image can be customized at start by using one of the following environment variables. If no default is mentioned, leaving the environement variable unspecified will not add a line to the properties file.

* `AXONSERVER_NAME`

    This is the name the Axon Server uses for itself.
* `AXONSERVER_HOSTNAME`

    This is the hostname Axon Server communicates to the client as its contact point. Default is "`localhost`", because Docker generates a random name that is not resolvable outside of the container.
* `AXONSERVER_DOMAIN`

    This is the domain Axon Server can suffix the hostname with.
* `AXONSERVER_HTTP_PORT`

    This is the port Axon Server uses for its Web GUI and REST API.
* `AXONSERVER_GRPC_PORT`

    This is the gRPC port used by clients to exchange data with the server.
* `AXONSERVER_TOKEN`

    Setting this will enable access control, which means the clients need to pass this token with each request.
* `AXONSERVER_EVENTSTORE`

    This is the directory used for storing the Events.
* `AXONSERVER_CONTROLDB`

    This is where Axon Server stores information of clients and what types of messages they are interested in.

### Axon Server configuration

There are a number of things you can finetune in the server configuration. You can do this using an "`axonserver.properties`" file. All settings have sensible defaults.

* `axoniq.axonserver.name`

    This is the name Axon Server uses for itself. The default is to use the hostname.
* `axoniq.axonserver.hostname`

    This is the hostname clients will use to connect to the server. Note that an IP address can be used if the name cannot be resolved through DNS. The default value is the actual hostname reported by the OS.
* `server.port`

    This is the port where Axon Server will listen for HTTP requests, by default `8024`.
* `axoniq.axonserver.port`

    This is the port where Axon Server will listen for gRPC requests, by default `8124`.
* `axoniq.axonserver.event.storage`

    This setting determines where event messages are stored, so make sure there is enough diskspace here. Losing this data means losing your Events-sourced Aggregates' state! Conversely, if you want a quick way to start from scratch, here's where to clean.
* `axoniq.axonserver.controldb-path`

    This setting determines where the message hub stores its information. Losing this data will affect Axon Server's ability to determine which applications are connected, and what types of messages they are interested in.
* `axoniq.axonserver.accesscontrol.enabled`

    Setting this to `true` will require clients to pass a token.
* `axoniq.axonserver.accesscontrol.token`

    This is the token used for access control.

### The Axon Server HTTP server

Axon Server provides two servers; one serving HTTP requests, the other gRPC. By default these use ports 8024 and 8124 respectively, but you can change these in the settings.

The HTTP server has in its root context a management Web GUI, a health indicator is available at `/actuator/health`, and the REST API at `/v1`. The API's Swagger endpoint finally, is available at `/swagger-ui.html`, and gives the documentation on the REST API.
