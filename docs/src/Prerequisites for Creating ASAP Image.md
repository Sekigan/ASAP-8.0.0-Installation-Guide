### Prerequisites for Creating ASAP Image

The prerequisites for building ASAP cloud native image are:
- Podman on the build machine.
- Approximately 2 GB of swap space on the machine where the Podman is running. By running the `free -m` command, you can verify the swap space.
    Note If the required swap space is not available, contact your administrator.
    
- ASAP 8.0.0.0 or later Linux Installer. Download the .tar file from Oracle Software Delivery Cloud:
    [https://edelivery.oracle.com](https://edelivery.oracle.com/)
    
- Create the disk1 directory and copy the contents of the .tar file to this directory.
- Installers for WebLogic Server and JDK. Download these from Oracle Software Delivery Cloud:
    [https://edelivery.oracle.com](https://edelivery.oracle.com/)
    
- Oracle Database Client. Download this from Oracle Software Downloads:
    [https://www.oracle.com/downloads/](https://www.oracle.com/downloads/).
    
- Release Updates 19.29.0.0.0 or later. Upgrade Oracle Database Client with Oracle Database Release Updates (RUs). Update the path of the patch list directory in the `installDBclient.sh` script located at `asap_img_builder/scripts`. For example, `$ORACLE_HOME/37257886/37260974`.
    
- OPatch utility version 12.2.0.1.34 or later.
    
- Java, installed with JAVA_HOME set in the environment.
    
- **ASAP is installed in a silent installation mode using the `asap_image.properties` file. You should update the properties file with the database, WebLogic Server, ORACLE_HOME, port numbers, and all required details.**
    
- **Keep the TRAEFIK Ingress service node port details ready where it is being deployed.**
    
- **Create ASAP database and database users. For more information, see "[Creating Oracle Database Tablespaces, Tablespace User, and Granting Permissions](https://docs.oracle.com/pls/topic/lookup?ctx=en/industries/communications/asap/8.0/cloud-native-deploy&id=ASPIG-GUID-F4F8FEAB-E65D-448E-850B-AF4193699A9A)" in ASAP Installation Guide.**

For details about the required and supported versions of the prerequisite software, see [ASAP Software Compatibility Matrix].