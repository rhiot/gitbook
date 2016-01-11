# Starting Cloud Platform

Reference Cloud Platform runs using Docker containers. We love Docker and believe that containers are the
future of the applications deployment. To install the Cloud Platform on the Linux server of your choice, just execute the
following command:

    bash <(curl -L -s https://goo.gl/nVsrg6)

The script above installs the proper version of Docker server. Keep in mind that the minimal Docker version required by
Cloud Platform is 1.8.2 - if the older version of the Docker is installed, our script will upgrade your Docker server. After
Docker server is properly installed, Cloud Platform script downloads and starts:

- protocol adapters
- default Rhiot backend services
- standalone Apache Spark cluster (a single master node and a single worker node)
- MongoDB database

Rhiot Cloud relies on the MongoDB to store some of the data processed by it. For example MongoDB backend is a default
store used by the device service. By default the MongoDB data is stored in the `mongodb_data`
volume container. If such volume doesn't exist, Cloud Platform script will create it for you.