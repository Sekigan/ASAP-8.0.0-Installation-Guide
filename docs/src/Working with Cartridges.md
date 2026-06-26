#### Working with Cartridges

An ASAP cartridge provides specific domain behavior on top of the core ASAP software. This domain behavior includes a part of, or all services on a network element (NE), element management system (EMS), or network management system (NMS).

To provision orders to network elements, you install cartridges in the ASAP container that are available in the `$ASAP_VOLUME/cartridges/` directory. After installing the cartridges, you must exit the ASAP container by using the `exit` command.

**To install cartridges:**

1. Copy the required cartridges to the `$asap-img-builder/cartridges` directory.
2. Run the following command to copy installers and cartridges to the volume:
    
    `$asap-img-builder/upgrade-asap-image.sh`
    
3. Create a new container with the ASAP image created by the `build-asap-images.sh` script using the following command:
    
    `podman run --name $ASAP_CONTAINER -dit -h $PODMAN_HOSTNAME -p $WEBLOGIC_PORT -v $ASAP_VOLUME:/$ASAP_VOLUME ASAP-BASE-IMAGE`
	
	`docker run --name $ASAP_CONTAINER -dit -h $DOCKER_HOSTNAME -p $WEBLOGIC_PORT -v $ASAP_VOLUME:/$ASAP_VOLUME ASAP-BASE-IMAGE`
    
    For example: `podman run --name asap-c -dit -h asaphost -p 7890 -v podmanhost_volume:/podmanhost_volume asapcn:8.0.0.0`
    **The container is created with asap-c.**
    
4. Enter the ASAP container using the following command:
      
    `podman exec -it CONTAINER_NAME bash`
    where `CONTAINER_NAME` is the $ASAP_CONTAINER. For example, asap-c.
    You have entered the ASAP container.
    
5. Start ASAP and WebLogic Server using the **`startALL.sh`** script.
   NOTE: SARM does not start successfully unless at least one cartridge is installed.
   During installation of the first cartridge, **after running the `startALL.sh` script, the console may display messages indicating that the ASAP servers did not start successfully. This is an expected behavior during the initial installation. You can ignore the messages** and the installation can proceed with the steps to follow.

6. Navigate to the ASAP installation directory using `cd $ASAP_BASE`.
7. Source the environment profile using the `source Environment_Profile` script.
8. Verify the ASAP server status using the `status` command.
9. Install the cartridges present in the `/podmanhost_volume/cartridges` directory.
10. Exit the ASAP container by using the following command:
    `exit`
11. After you install the cartridge, create an image from the staging container using the following command:
    `podman commit CONTAINER_NAME imagename:version`
    `docker commit CONTAINER_NAME imagename:version`
    Where
	- `CONTAINER_NAME` is the $ASAP_CONTAINER.
	- `version` is the version of the ASAP image.
	  
12. Stop and remove the containers using the following commands:
	`podman stop CONTAINER_NAME`
	`docker stop CONTAINER_NAME`
	where `CONTAINER_NAME` is the $ASAP_CONTAINER.