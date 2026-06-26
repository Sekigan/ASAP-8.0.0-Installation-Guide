Order Balancer cloud native image is required to create and manage Order Balancer cloud native instances. This chapter describes creating an Order Balancer cloud native image.

Order Balancer cloud native instance requires a **container image and access to the database**. The Order Balancer image is built on top of a Linux base image and the Order Balancer image builder script adds Java, WebLogic Server components, and Order Balancer.

The Order Balancer cloud native image is created using the Order Balancer cloud native builder toolkit. You should run the Order Balancer cloud native builder toolkit on Linux and it should have access to the local Podman.

### [[Downloading the ASAP Cloud Native Artifacts]]
The Order Balancer cloud native builder kit contains:

- The scripts to install the required packages.
- The scripts to install database client, install Java, WebLogic Server, and ASAP.

### [[Prerequisites for Creating an Order Balancer Image]]

### Creating the Order Balancer Cloud Native Image

The Order Balancer cloud native image builder tool builds the Order Balancer cloud native image, which is then pushed to the repository and deployed in the Kubernetes cluster. If the repository is not available, you copy the image to all the worker nodes of the cluster.

The ASAP installer is packaged with the Order Balancer cloud native image builder and the cloud native toolkit.
Note: After you download the installer, locate the Order Balancer cloud native image builder `asap-img-builder.zip` in the ASAP cloud native tar file.

To create the Order Balancer cloud native image:

1. Copy the `asap-img-builder.zip` file to the machine where the Podman is running.
2. Extract the contents of the zip file by running the following command:
    `unzip asap-img-builder.zip`
3. Copy the following installers to the `$asap-img-builder/installers` directory.
    - Order Balancer Installer
    - Installers for WebLogic Server and JDK
    - ANT 1.10.15
    - JDK 21
4. Update the following parameters in the `$asap-img-builder/ob.properties` file:
```bash
ob.tar.file=ASAP.R8_0_0.Bxxx.ob.tar
ob.weblogic.username=weblogic
ob.weblogic.password=
ob.weblogic.port=7501
ob.weblogic.domainName=ob
ob.weblogic.channel.listenport=7502
ob.weblogic.channel.publicport=30305
ob.ssl.incoming=0
ob.cacheexpiry=60
#Time in seconds
ob.all.servers.down.wait.interval=3600
ob.all.servers.down.retry.interval=120
ob.server.down.retry.interval=2
ob.server.poll.interval=60
ob.webservice.res.timeout=0
ob.asap.conn.timeout=10
## Values allowed: SEVERE, WARNING, INFO , FINE , FINEST ,ALL
ob.logger.info=INFO
ob.db.host=
ob.db.port=1521
ob.db.service.name=
ob.db.user=
ob.db.password=
ob.jms.user=
ob.jms.password=
#RAC Database TNS parameters 
ob.db.failover.mode.type=SELECT
ob.db.failover.mode.method=PRECONNECT 
ob.db.failover.retries=20 
ob.db.failover.delay=3
ob.db.backup.node=X.X.X.X
```

where

- `ob.weblogic.username` is the user name to log in to WebLogic Server.
- `ob.weblogic.password` is the password to log in to WebLogic Server.
- `ob.weblogic.port` is the port of the WebLogic Server.
- `ob.weblogic.domainName` is the WebLogic Server domain.
- `ob.weblogic.channel.listenport` is the channel listen port of the WebLogic Server.
- `ob.weblogic.channel.publicport` is the public port of the WebLogic Server.
- `ob.ssl.incoming` is set to enable SSL on Order Balancer WebLogic Server. The default value is 0 which specifies non-SSL.
- `ob.cacheexpiry` specifies the duration in seconds that Order Balancer JPA shared query refreshes the cache. Cache is refreshed upon next request after the duration expires. The default value is 60.
- `ob.all.servers.down.wait.interval` specifies the duration in seconds that Order Balancer waits before routing the request back to queue when all the ASAP instances are down. The default value is 3600.
- `ob.all.servers.down.retry.interval` specifies the duration in seconds that Order Balancer waits before retrying to connect to fetch for an active ASAP member instance while waiting when all servers are down. The default value is 120.
- `ob.server.down.retry.interval` specifies the duration in seconds that Order Balancer waits before reattempting to route the order to the same instance. If the re-attempt fails, the instance is marked as down. The default value is 2.
- `ob.server.poll.interval` specifies the duration in seconds that Order Balancer waits before it retries to check the ASAP instance status. The default value is 60.
- `ob.webservice.res.timeout` specifies the duration in seconds that Order Balancer waits for response before the read times-out. Order Balancer Web Service waits for a response from ASAP member instance after invoking the operation. A value of zero means Order Balancer will wait indefinitely until it receives a response from ASAP. The default value is 0 seconds (no read time-out).
- `ob.asap.conn.timeout` specifies the duration in seconds that Order Balancer reattempts the connection to the ASAP instance. The default value is 10.
- `ob.logger.info` specifies the log level for initializing the Order Balancer application root logger. The valid values are SEVERE, WARNING, INFO, FINE, FINEST, and ALL.
- `ob.db.host` is the database host name or IP address.
- `ob.db.port` is the database port.
- `ob.db.service.name` is the database service name.
- `ob.db.user` is the database user name.
- `ob.db.password` is the database password.
- `ob.jms.user` is the JMS user.
- `ob.jms.password` is the JMS password.
- `ob.db.failover.mode.type` specifies the type of failover to perform when the primary database node fails. Valid values are PRECONNECT which pre-establishes a backup connection for faster failover and BASIC which connects to the backup only after a failure occurs. The default value is Select.
- `ob.db.failover.mode.method` specifies how the failover connection is established. Valid values are PRECONNECT and BASIC. The default value is PRECONNECT.
- `ob.db.failover.retries` specifies the number of retry attempts to connect to the backup node in case of failure. The default value is 20.
- `ob.db.failover.delay` specifies the delay in seconds between each failover retry attempt. The default value is 3.
- `ob.db.backup.node` specifies the IP address or hostname of the backup database node used for failover. If the multidata source is not used, then the parameter value should be "none".

Note: **Do not add an ASAP instance when you are building the Order Balancer image**. The wallet store is mounted dynamically in the Kubernetes cluster. The wallet files created in the image are not accessible in the Kubernetes Pod.

**In the cloud native deployment, the WebLogic domain is non-SSL and the ingress controller is configured as SSL.**

5. Update the `HTTPS_PROXY` and `HTTP_PROXY` variables in the `build_ob_env.sh` script:
```bash
base_image=oraclelinux:8
HTTPS_PROXY=
HTTP_PROXY=

# Podman details
OB_IMAGE_TAG="obcn:8.0.0.0.0"
OB_VOLUME=obhost_volume
OB_CONTAINER="ob-c"
PODMAN_HOSTNAME="obhost"

# Installer filenames
WEBLOGIC_DOMAIN=/u01/oracle/user_projects/domains/
JDK_FILE=jdk-21_linux-x64.tar.gz
FMW_FILE=fmw_V1045131-01.zip

# Installation locations
JAVA_HOME=/usr/lib/jvm/java/jdk-21.0.9
PATH=$JAVA_HOME/bin:$PATH
WEBLOGIC_HOME=/home/oracle/weblogic1412
```
   
Note: The file names of JDK_FILE and FMW_FILE variables must match with the file names in the `/asap-img-builder/installers/` folder.

6. Run the `build-asap-images.sh` script to build the Order Balancer images:
    `./build-asap-images.sh -i ob`
    
    The script creates the Order Builder images by running Podman and commits the Order Builder image.