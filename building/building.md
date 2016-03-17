# Building Rhiot

All you need to build the project is Maven 3 and Java 8 JDK

    git clone https://github.com/rhiot/rhiot.git
    cd rhiot
    mvn install

## Building project using Docker

If you don't want to install Java and Maven on your machine, you can use our Docker building image containing all the
tools you need.

    git clone https://github.com/rhiot/rhiot.git
    cd rhiot
    docker run -v `pwd`:/rhiot --privileged -it rhiot/build

## Releasing the project

If you are interested in cutting a release of the project follow steps described below:

### Before you start

You have to had a DockerHub account setting configured in your `~/.m2/settings.xml`:
    
    <server>
      <username>rhiot</username>
      <password>secret</password>
      <id>registry.hub.docker.com</id>
    </server>

### Cutting the release

Execute the following command in the project main directory:

    mvn release:prepare release:perform

### After the release

* upgrade the version on the main page of the project ([readme.md](https://github.com/rhiot/rhiot/blob/master/readme.md))
* update release guide [Releases Notes](../releases_notes/releases_notes.md) using GitHub tickets marked as done in the given version
* upgrade version in Rhiot command line (`tooling/bash/rhiot.sh`). Look up for a line similar to `RHIOT_VERSION=x.y.z`.
* upgrade Rhiot version in quickstarts (use the latest stable version).
* upgrade Rhiot version of Rhiot cloud (use the latest stable version) - modify `RHIOT_VERSION=` line in a [rhiot-cloud-platform](https://github.com/rhiot/rhiot/blob/master/cloudplatform/install/rhiot-cloud-platform) file.
* upgrade Rhiot version in [Docker images](https://github.com/rhiot/rhiot/tree/master/dockerfiles) not managed by Docker Maven Plugin