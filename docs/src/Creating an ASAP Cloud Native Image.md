# Creating an ASAP Cloud Native Image
#### Downloading the ASAP Cloud Native Image Builder
To build the ASAP cloud native image, the `asap-img-builder.zip` file is required. For more information about downloading the ASAP cloud native Image Builder, see "[[Downloading the ASAP Cloud Native Artifacts]]".

ASAP cloud native builder kit contains:

- The scripts to install the required packages.
- The scripts to install database client, Java, WebLogic Server, Opatch, Release Updates (RU) and ASAP.

#### [[Prerequisites for Creating ASAP Image]]

#### Creating the ASAP Cloud Native Image

The ASAP cloud native image builder tool builds the ASAP cloud native image, which is then pushed to the repository and deployed in the Kubernetes cluster. If the repository is not available, you copy the image to all the worker nodes of the cluster.

The ASAP installer is packaged with the ASAP cloud native image builder and the cloud native toolkit.
To create the ASAP cloud native image:

1. Copy the `asap-img-builder.zip` file to the machine where the Podman is running.
2. Extract the contents of the zip file by running the following command:
    `unzip asap-img-builder.zip`
3. Copy the following installers to the `$asap-img-builder/installers` directory.
    - Linux Installer for ASAP 8.0 or later
    - Installers for WebLogic Server.
    - Oracle Database Client
    - OPatch Utility
    - JDK 21
    - Oracle Database Release Updates
    - Redhat Package Manager(RPM) Installer
4. **Copy the required cartridges to the `$asap-img-builder/cartridges` directory.**
5. Set the environment variable for ASAP_IMG_BUILDER to the `$asap-img-builder` directory.
6. Update the `$asap-img-builder/scripts/tnsnames.ora` file with the database details. For example,
```bash
orcl19c =
   (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = <database_host>)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = service name)
    )
  )
```
7. Update the `HTTPS_PROXY` and `HTTP_PROXY` variables in the `build_env.sh` script.
```env
base_image=100.76.142.126:5000/oraclelinux:8
HTTPS_PROXY=http://www-proxy-brmdc.us.oracle.com:80
HTTP_PROXY=http://www-proxy-brmdc.us.oracle.com:80

# Podman details
ASAP_IMAGE_TAG="8.0.0.0.0"
ASAP_VOLUME=podmanhost_volume_c0
ASAP_CONTAINER="asap-wr73"
PODMAN_HOSTNAME="asaphost"

# Installer filenames
JDK_FILE=jdk-21_linux-x64_bin.tar.gz
DB_CLIENT_FILE=LINUX.X64_193000_client.zip
FMW_FILE=V1045131-01.zip
DB_OPATCH_FILE=p6880880_190000_Linux-x86-64.zip
DB_RU_FILE=p35319490_190000_Linux-x86-64.zip
RPM_FILE=asap-installer-8.0.0.0.0-B92.x86_64.rpm
ANT_FILE=apache-ant-1.10.15-bin.tar.gz

## Installation locations
TNS_ADMIN=/scripts/
JAVA_HOME=/usr/lib/jvm/java/jdk-21.0.9
PATH=$ORACLE_HOME:$JAVA_HOME/bin:$PATH
CV_ASSUME_DISTID=OEL8
```
JDK 21 is required by the ASAP installer and the RPM installer.
NOTE: Ensure that the file names of `JDK_FILE`,`DB_CLIENT_FILE`, `FMW_FILE`, `DB_OPATCH_FILE`, `DB_RU_FILE`, and `RPM_FILE` variables match with the file names in the `$ASAP_IMG_BUILDER/installers` directory.


<!-- Help with setting parameters  -->
//`I need help configuring the parameters.`
8. Update the following parameters in the `$asap-img-builder/asap_image.properties` file with the required details for ASAP and the WebLogic Server domain. You can update the configuration parameters that must be updated after ASAP installation using `asap_image.properties` file. 
   All the configuration parameters should be prefixed with `cfg`. You can create multiple ASAP instances by entering multiple unique environment IDs in the `ENV_ID` variable separated by a comma. For example, `ENV_ID=`CNE1, CNE2, CNE3 where CNE1, CNE2, and CNE3 are unique environment IDs.
```env
##########################################################################
##  Configure ASAP LINUX Environment Variables
##########################################################################
##  Type the name of the base directory of the Oracle database client program. The ASAP installation generates an ORACLE_HOME LINUX environment variable based on the directory name.
ORACLE_HOME=
##  Select the ASAP mode that you require. You can run ASAP in the Production (PROD) or Development (TEST) mode. ASAP loads static provisioning configuration information from the database based on mode. 
ASAP_SYS=TEST
##  Type the ASAP environment ID (maximum 4 alphanumeric characters). The ASAP environment ID is a unique identifier for each ASAP environment as one system  can have multiple ASAP instances.
ENV_ID=
##  Type the name of the location where the ASAP needs to be installed in the image
ASAP_BASE=


##########################################################################
##  Configure ASAP Database 
##########################################################################
##  The ASAP installation program retrieves the name of the Oracle RDBMS Server from tnsname.ora file on the installed Oracle client program. 
##  Name of Oracle RDBMS Server :
DSQUERY=
##  Oracle Server DBA User Name  :
DB_USER=
##  Oracle Server DBA Password  :
DB_PASSWORD=
##                         ASAP Database Users Creation
##                        ------------------------------
##  CREATE_USERS=true:  The database users will be created during the ASAP configuration.
##  CREATE_USERS=false: The database users must be created by the user beforehand.
CREATE_USERS=true
##  Is this an Oracle RAC database :
RACDB=false
##  RAC Database Connection String, Format: Host1:Port1:ServiceName,Host2:Port2:ServiceName,... :
RACDB_CONNECT_STRING=

##########################################################################
##  Configure ASAP Database Table spaces 
##########################################################################
##Specify the required table space for each ASAP server. The recommended table space set up for the SARM server is SARM_DATA for the data table space and SARM_INDEX for the index table space. 
#The recommended table space for all other ASAP servers is DATA for data table space and INDX for index table space. ASAP database scripts populate the database schema based on the defined values. 
##  ADM
ADMDB_PARAMETERS=ADM, POOL_TS, POOL_TS
##  Control
CTRLDB_PARAMETERS=CTRL, POOL_TS, POOL_TS
##  NEP
NEPDB_PARAMETERS=NEP, POOL_TS, POOL_TS
##  SARM
SARMDB_PARAMETERS=SARM, POOL_TS, POOL_TS
##  TEMP ( Specify the temporary table space you want ASAP to use.)
TEMP_TS=TEMP

##########################################################################
## Configure ASAP Database User Password 
##########################################################################
##For each ASAP server, type the password for each database schema. ASAP database scripts create user schemas based on the user name and password that you define. 
##  ADM
ADMDB_PASSWORD=
##  Control
CTRLDB_PASSWORD=
##  NEP
NEPDB_PASSWORD=
##  SARM
SARMDB_PASSWORD=

##########################################################################
##  Configure ASAP LINUX Environment Variables - ASAP Server Port
##########################################################################
##Define the port number for ASAP servers.
##  CTRL Server
CTRL_PORT=30001
##  SARM Server
SARM_PORT=30002
##  NEP Server
NEP_PORT=30003
##  Define the port number for work order event notification for the OCA SRP. This information will be populated in tbl_asap_srp table in the SARM database.  
## Oca Server
OCA_PORT=30004
##  Define the port number for the Java SRP server to send work orders. This information will be populated in the tbl_listens table in the Control database. Define the port number for the Java SRP server to receive work order events. This information will be populated in the tbl_asap_srp table in the SARM database.  
##  JSRP - Sending WO
JSRPsend_PORT=30005
##   JSRPS123 - Receiving WO event 
JSRPrecev_PORT=30006
##   ASAP DAEMON 
DAEMON_PORT=30007
##   JeNEP Listener Port 
NEP_Listener_PORT=30008

##########################################################################
##  Configure Oracle WebLogic Server for ASAP
##########################################################################
##  Path to the WebLogic installation directory
WL_HOME=/home/oracle/weblogic
##  User Name of the Oracle WebLogic Server Administrator
WLS_USER=
##  HOST for the Oracle WebLogic Server
WLS_HOST=
##      Port for the Oracle WebLogic AdminServer
##  ------------------------------------------------
##  Specify NON-SSL Port if WLS_SSL_ENABLED is false
##  Specify SSL Port if WLS_SSL_ENABLED is true 
WLS_ADMIN_PORT=7890
##  NON SSL Port for the target server of the WebLogic
WLS_TARGET_PORT=7890
##  SSL Port for the target server of the WebLogic
WLS_TARGET_SSL_PORT=
##  Oracle WebLogic Server
WLS_SERVER=AdminServer
## Use SSL?
WLS_SSL_ENABLED=true
## SSL KeyStore File
KEYSTORE_FILE=
WLS_CUSTOM_TRUST_KEYSTORE_TYPE=pkcs12
WLS_CUSTOM_TRUST_PASSPHRASE=
MAX_QUERY_LIMIT=200
CFG.DB_ADMIN_ON=1
CFG.DB_ADMIN_PROC_PARAM=30
## Weblogic Domain Name
WLS_DOMAIN_NAME=asapDomain
## Weblogic Domain Location 
WLS_DOMAIN_LOCATION=
## Weblogic Channel Listen Port
WLS_CHANNEL_LISTEN_PORT=7891
## Weblogic Channel Public Port
WLS_CHANNEL_PUBLIC_PORT=30305


##########################################################################
## Configure Oracle WebLogic Server Passwords
##########################################################################
##  Each password must be at least 8 characters long, and must contain at least 1 number or special character.
##  ASAP_admin
ADMIN_PASSWORD=
##  cmws_studio
CMWS_PASSWORD=
##  ASAP_monitor
MONITOR_PASSWORD=
##  ASAP_operator
OPERATOR_PASSWORD=
##  WebLogic admin password
WLS_PASSWORD=
##  asap_ws_user
WS_PASSWORD=

##########################################################################
## Configure GRPC specific properties
##########################################################################
## SSL for GRPC ? 1 = Enabled, 0 = Not Enabled
GRPC_SSL_ENABLED=0
## Authentication for GRPC ? 1 = Enabled, 0 = Not Enabled
GRPC_AUTH_ENABLED=0
## Absolute path of SSL identity certificate in the pkcs12 format
SSL_IDENTITY_PKCS12_LOCATION=
## Absolute path of SSL trust certificate in the pkcs12 format
SSL_TRUST_PKCS12_LOCATION=
##########################################################################
## Configure ASAP cfg values
##########################################################################

CFG.MSGSND_RETRIES=6
CFG.DB_ADMIN_ON=1
CFG.DB_ADMIN_PROC_PARAM=30
```
<!-- Help with setting parameters  -->
NOTE: 
- You must not change the default value (`0 - Not Enabled`) for the parameters `GRPC_SSL_ENABLED` and `GRPC_AUTH_ENABLED`.
- If Traefik is configured, `WLS_CHANNEL_PUBLIC_PORT` parameter must have the traefik ingress controller nodeport value (for http 30305 and for https 30443).
- If DNS is configured, `WLS_CHANNEL_PUBLIC_PORT` must have the value same as `WLS_CHANNEL_LISTEN_PORT`.
- `ADMIN_PASSWORD`, `CMWS_PASSWORD`, `MONITOR_PASSWORD`, `OPERATOR_PASSWORD`, `WLS_PASSWORD`, and `WS_PASSWORD` parameters must have passwords at least 8 characters long, and must contain at least 1 number or special character.
- The `asap_image.properties` file is available only in the host machine and not as part of the ASAP image.
<!-- Help with setting parameters  -->


9. Run the `build-asap-images.sh` script using the following command:
   **`./build-asap-images.sh -i asap`**
   
   The script creates the staging ASAP image for the environment IDs specified in the `asap_image.properties` file by installing WebLogic Server, Java, and the database client.


### [[Working with Cartridges]]

### Securing Your ASAP Installation

ASAP security is designed to provide confidentiality, data integrity, and ensure on-demand access to services for authorized users. For information about ASAP security, see "[Setting Up and Managing ASAP Security](https://docs.oracle.com/pls/topic/lookup?ctx=en/industries/communications/asap/8.0/cloud-native-deploy&id=ASPSA-GUID-2184A032-B068-4790-B92C-6C321C5990DB)" in ASAP System Administrator's Guide.
