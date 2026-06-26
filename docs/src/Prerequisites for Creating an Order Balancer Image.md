### Prerequisites for Creating an Order Balancer Image

The prerequisites for building an Order Balancer cloud native image are:

- Podman on the build machine.
- Approximately 2 GB of swap space on the machine where the Podman is running. By running the `free -m` command, you can verify the swap space.
	Note: If the required swap space is not available, contact your administrator.
- Order Balancer Installer file. For example, `ASAP.R8_0_0_Px.Byy.ob.tar` file. Download this file from the Oracle Software Delivery Cloud website:[https://edelivery.oracle.com](https://edelivery.oracle.com/).
- Installers for WebLogic Server and JDK. Download these from the Oracle Software Delivery Cloud website: [https://edelivery.oracle.com](https://edelivery.oracle.com/).
- Java, installed with `JAVA_HOME` set in the environment.
- TRAEFIK Ingress service node port should be ready where it is being deployed.
- **Order Balancer database users should be created.** For more information, see "[[Planning you installation Order Balancer]]" in ASAP System Administrator's Guide.
