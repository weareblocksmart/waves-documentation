# Waves

The latest release for each network can be found in the [Releases section](https://github.com/wavesplatform/Waves/releases), you can switch to the corresponding tag and build the application.

[How to configure Waves node](/waves-full-node/how-to-configure-a-node.md)

# Installation

Please read the [Waves Node Installation guide](/waves-full-node/how-to-install-a-node/how-to-install-a-node.md).

## Compiling Packages from source

It is only possible to create deb and fat jar packages.

### Install SBT \(Scala Build Tool\)

For Ubuntu/Debian:

```js
echo "deb https://dl.bintray.com/sbt/debian /" | sudo tee -a /etc/apt/sources.list.d/sbt.list
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 2EE0EA64E40A89B84B2DF73499E82A75642AC823
sudo apt-get update
sudo apt-get install sbt
```

You can install sbt on Mac OS X using Homebrew.

### Create Package

Clone this repo and execute

```js
sbt packageAll
```

.deb and .jar (the correct JAR-file has name `waves-all-*.jar`) packages will be in /package folder. To build testnet packages use

```js
sbt -Dnetwork=testnet packageAll
```

# Running Tests

`sbt test`

**Note**

If you prefer to work with\_SBT\_in the interactive mode, open it with settings:

```js
SBT_OPTS="${SBT_OPTS} -Xms512M -Xmx1536M -Xss1M -XX:+CMSClassUnloadingEnabled" sbt
```

For Java9 it should be:

```js
SBT_OPTS="${SBT_OPTS} -Xms512M -Xmx1536M -Xss1M -XX:+CMSClassUnloadingEnabled --add-modules=java.xml.bind --add-exports java.base/jdk.internal.ref=ALL-UNNAMED" sbt
```

to solve the`Metaspace error`problem.

# Running Integration Tests

## TL;DR

* Make sure you have [Docker](https://www.docker.com/get-docker) and SBT.
* `sbt it/test`

## Customizing Tests

By default,`it/test` will do the following:

* Build a container image with the fat jar and a [template.conf](https://github.com/wavesplatform/Waves/blob/master/src/it/resources/template.conf). The newly-built image will be registered with the local Docker daemon. This image is built with [sbt-docker](https://github.com/marcuslonnberg/sbt-docker) plugin.
* Run the test suites from `src/it/scala`, passing docker image ID via `docker.imageId` system property.

### Logging

By [default](https://github.com/wavesplatform/Waves/blob/master/src/main/resources/logback.xml) all logs are written to the STDOUT. If you want to write logs, for example, to JSON files, you should define your own logging configuration and specify a path to it in`conf/application.ini`:

```
-Dlogback.configurationFile=/path/to/your/logback.xml
```

### Debugging

Integration tests run in a forked JVM. To debug test suite code launched by SBT, you will need to add remote debug options to`javaOptions`in`IntegrationTest`configuration:

```js
javaOptions in IntegrationTest += "-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005"
```

Debugging a node inside a container is a little more complicated: you will need to modify the`WAVES_OPTS`environment variable before starting a container.

### Running Tests from IDE

You can run integration test suites from your preferred IDE. The only requirement is to have Docker image pre-built and have`docker.imageId`system property defined for the run configuration. The easiest way to build an image is to issue`sbt docker`command. You'll find the image ID in the SBT output:

```js
...
[info] Step 5/5 : ENTRYPOINT /opt/waves/start-waves.sh
[info]  ---> Using cache
[info]  ---> e243fa08d496
[info] Successfully built e243fa08d496
[info] Tagging image e243fa08d496 with name: com.wavesplatform/root
[success] Total time: 4 s, completed Mar 22, 2017 12:36:34 PM
```

In this example,`e243fa08d496`is the image ID you need. Make sure to re-build the image whenever the node code \(not the tests\) is changed. If you run the tests from SBT, there's no need to manually rebuild the image, SBT will handle this automatically.

# Collecting performance metrics

**Note**: all required tools will be installed though [Docker](https://docs.docker.com/) for simplicity.

* Install [Graphite](https://graphite.readthedocs.io/en/latest/install.html#docker), a service for collecting metrics.

* Install [Grafana](https://grafana.com/grafana/download?platform=docker) for beautiful graphs.

By default all metrics are disabled. So specify\_Kamon\_settings through\_Java Properties\_and run the node with a desired config. For example, we ran\_Graphite\_locally and it accepts\_StatsD\_information on the`9999`port:

```js
SBT_OPTS="${SBT_OPTS} -Xms512M -Xmx1536M -Xss1M -XX:+CMSClassUnloadingEnabled \
-Dkamon.modules.kamon-statsd.auto-start=yes \
-Dkamon.modules.kamon-system-metrics.auto-start=yes \
-Dkamon.statsd.hostname=localhost \
-Dkamon.statsd.port=9999" sbt waves-testnet.conf
```

Here:

* `-Dkamon.modules.kamon-statsd.auto-start=yes` enables custom metrics;
* `-Dkamon.modules.kamon-system-metrics.autostart=yes` enables metrics of CPU, Memory and others;
* See [application.conf](https://github.com/wavesplatform/Waves/blob/master/src/main/resources/application.conf) for more options.
