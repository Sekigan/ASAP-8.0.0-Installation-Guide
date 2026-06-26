### Next Steps

The ASAP instance is ready to add the Order Balancer cloud native instance. The URL for adding the Order Balancer instance is:

`t3://service-name.namespace.svc.cluster.local:portnumber`

Where:

- `service-name` is the name of the service.
- `namespace` is the namespace of the ASAP instance.
- `portnumber` is the port number of the WebLogic Server Domain in the ASAP instance.

Here is an example for adding the ASAP instance to Order Balancer:

- `./addASAPServer -asapSrvName ASAP1 -asapSrvURL t3://cne1-service.sr.svc.cluster.local:7890 -asapSrvUser weblogic -asapSrvRequestQueue "System.CNE1.ApplicationType.ServiceActivation.Application.1-0;8-0;ASAP.Comp.MessageQueue"`