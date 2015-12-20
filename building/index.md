# Building Rhiot

All you need to build the project is Maven 3 and Java 8 JDK

    git clone https://github.com/rhiot/rhiot.git
    cd rhiot
    mvn install

## Building project using Docker

If you don't want to install Java and Maven on your machine, you can use our Docker building image.

### 1st step (once)

    git clone https://github.com/rhiot/rhiot.git
    cd rhiot/dockerfiles
    docker build -t rhiot build


### 2nd step

    cd rhiot
    docker run -v `pwd`:/rhiot --privileged -i -t rhiot

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

* Update the version on the main page of the project [readme.md](https://github.com/rhiot/rhiot/blob/master/readme.md))
* Update release guide [Releases Notes](../releases_notes/index.md) using GitHub tickets marked as done in the given version
* upgrade version in Rhiot command line (`tooling/bash/rhiot.sh`). Look up for a line similar to `RHIOT_VERSION=x.y.z`.
