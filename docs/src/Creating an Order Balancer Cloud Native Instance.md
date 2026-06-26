This chapter describes how to create an Order Balancer cloud native instance in your cloud environment using the operational scripts and the base Order Balancer configuration provided in the Order Balancer cloud native toolkit.

Before you create an Order Balancer instance, you must do the following:

- Download the ASAP cloud native tar file and extract the `ob-cntk.zip` file. For more information about downloading the Order Balancer cloud native toolkit, see "[[Downloading the ASAP Cloud Native Artifacts]]".

### Installing the Order Balancer Artifacts and the Toolkit

Build container image for the Order Balancer application using the Order Balancer cloud native Image Builder.
You must create a private repository for this image, ensuring that all nodes in the cluster have access to the repository. See "[About Container Image Management](https://docs.oracle.com/en/industries/communications/asap/8.0/cloud-native-deploy/planning-validating-your-cloud-environment1.html#GUID-C071DB90-B391-44AB-BA7B-65CD0FEC8D52)" for more details.
Copy the Order Balancer cloud native toolkit that is the `ob-cntk.zip` file to one of the nodes in the Kubernetes cluster and do the following:
- On Oracle Linux: Where Kubernetes is hosted on Oracle Linux, download and extract the tar archive to each host that has connectivity to the Kubernetes cluster.
- On OKE: For an environment where Kubernetes is running in OKE, extract the contents of the tar archive on each OKE client host. The OKE client host is the bastion host/s that is set up to communicate with the OKE cluster.

Set the variable for the installation directory by running the following command:
`$ export OB_CNTK=ob_cntk_path`

In the previous example, `ob_cntk_path` is the installation directory of the Order Balancer cloud native toolkit.

### Installing the Traefik Container Image

To leverage the Order Balancer cloud native samples that integrate with Traefik, t**he Kubernetes environment must have the Traefik ingress controller installed and configured**.

If you are working in an environment where the Kubernetes cluster is shared, confirm whether Traefik has already been installed and configured for Order Balancer cloud native. If Traefik is already installed and configured, set your `TRAEFIK_NS` environment variable to the appropriate name space.

The instance of Traefik that you installed to validate your cloud environment must be removed as it does not leverage the Order Balancer cloud native samples. Ensure that you have removed this installation in addition to purging the Helm release. Check that any roles and rolebindings created by Traefik are removed. There could be a clusterrole and clusterrolebinding called "traefik-operator". There could also be a role and rolebinding called "traefik-operator" in the $TRAEFIK_NS name space. Delete all of these before you set up Traefik.
```bash
kubectl get clusterrole -n $TRAEFIK_NS 
kubectl get clusterrolebinding -n $TRAEFIK_NS 
kubectl delete clusterrole <clusterrole_name> -n $TRAEFIK_NS 
kubectl delete clusterrolebinding <clusterrolebinding_name> -n $TRAEFIK_NS 
```

To download and install the Traefik container image:

1. Ensure that Podman in your Kubernetes cluster can pull images from the container registry. See [ASAP Compatibility Matrix] for the required and supported versions of the Traefik image.
2. Run the following command to create a name space ensuring that it does not already exist:
    Note: You might want to add the traefik name space to the environment setup such as `.bashrc`.
```bash
kubectl get namespaces
export TRAEFIK_NS=traefik    
kubectl create namespace $TRAEFIK_NS
```

3. Run the following commands to install Traefik using the `$OB_CNTK/samples/charts/traefik/values.yaml` file in the samples:
```bash
helm repo add traefik https://traefik.github.io/charts
helm install traefik-operator traefik/traefik \  
--namespace $TRAEFIK_NS \  
--values $OB_CNTK/samples/charts/traefik/values.yaml \   
--set "kubernetes.namespaces={$TRAEFIK_NS}"   
--version 26.0.0`
```
In the example, repo_link is [https://helm.traefik.io/traefik](https://helm.traefik.io/traefik) or [https://traefik.github.io/charts](https://traefik.github.io/charts) based on the helm chart version. For more details, see: [https://github.com/traefik/traefik-helm-chart](https://github.com/traefik/traefik-helm-chart)

After the installation, Traefik monitors the name spaces listed in its `kubernetes.namespaces` field for Ingress objects. The scripts in the toolkit manage this name space list as part of creating and tearing down Order Balancer cloud native projects.

When the `values.yaml` Traefik sample in the Order Balancer cloud native toolkit is used as is, Traefik is exposed to the network outside of the Kubernetes cluster through port 30305. To use a different port, edit the YAML file before installing Traefik. Traefik metrics are also available for Prometheus to scrape from the standard annotations.

Traefik function can be viewed using the Traefik dashboard. Create the Traefik dashboard by running the instructions provided in the `$OB_CNTK/samples/charts/traefik/traefik-dashboard.yaml` file. To access this dashboard, the URL is: `http://traefik.asap.org`. This is if you use the `values.yaml` file provided with the Order Balancer cloud native toolkit; it is possible to change the hostname as well as the port to your desired values.

### Creating an Order Balancer Instance

This section describes how to create an Order Balancer instance.
#### Setting Environment Variables
ASAP cloud native relies on access to certain environment variables to run seamlessly. Ensure the following variables are set in your environment:
- Path to your private specification repository
- Traefik name space

To set the environment variables:
- Set the `TRAEFIK_NS` variable for Traefik name space as follows:
    `$ export TRAEFIK_NS=traefik`

#### [[Creating Secrets]]
You must store sensitive data and credential information in the form of Kubernetes Secrets** that the scripts and Helm charts in the toolkit consume. Managing secrets is out of the scope of the toolkit and must be implemented while adhering to your organization's corporate policies. Additionally, **ASAP cloud native does not establish password policies.

#### Registering the Namespace

After you set the environment variables, register the name space. To register the name space, run the following command:
```bash
$OB_CNTK/scripts/register-namespace.sh -p sr -t targets 
# For example,
$OB_CNTK/scripts/register-namespace.sh -p sr -t traefik
```
Where `-p` is the namespace where Order Balancer is being deployed.

Note: `traefik` is the name of the targets for registration of the namespace sr. The script uses `TRAEFIK_NS` to find these targets. Do not provide the `traefik` target if you are not using Traefik.


### Creating an Order Balancer Instance
This procedure describes how to create an Order Balancer instance in your environment using the scripts that are provided with the toolkit.

To create an Order Balancer instance:
1. Update the `$OB_CNTK/charts/asap/values.yaml` file with the following values:
```yaml
replicaCount: 1
image:
  repository: obcn
  pullPolicy: IfNotPresent
  tag: "8.0.0.0.0"
imagePullSecrets:
  - name: asap-imagepull
obEnv:
  envid: ob96
  port: 7501
  host: obhost
  domain: ob
persistence:
  enabled: true

nameOverride: ""
fullnameOverride: ""

serviceAccount:
  # Specifies whether a service account should be created
  create: true
  # Annotations to add to the service account
  annotations: {}
  # The name of the service account to use.
  # If not set and create is true, a name is generated using the fullname template
  name: ""

podAnnotations: {}


podSecurityContext: {}
  # fsGroup: 2000

securityContext: {}
  # capabilities:
  #   drop:
  #   - ALL
  # readOnlyRootFilesystem: true
  # runAsNonRoot: true
  # runAsUser: 1000
servicechannel:
  name: channelport
  type: ClusterIP
  port: 7502

service:
  name: adminport
  type: ClusterIP
  port: 7501

ingress:
  type: TRAEFIK
  enabled: true
  sslIncoming: false
  adminsslhostname: adminobhostssl.asap.org
  adminhostname: adminobhost.asap.org

  secretName: project-instance-obcn-tls-cert

  hosts:
    - host: adminobhost.asap.org
      paths: []
  tls: []
  #  - secretName: chart-example-tls
  #    hosts:
  #      - chart-example.local

resources:
 requests:
  memory: "4Gi"
  cpu: "2"
 limits:
  memory: "4Gi"
  cpu: "2"

autoscaling:
  enabled: false

nodeSelector: {}

tolerations: []

# To avoid co-locating two OB replicas/pods on the same node remove {} and 
# uncomment below lines. Replace <OB-ENV-ID> with OB Env ID
affinity: {}
#  podAntiAffinity:
#    requiredDuringSchedulingIgnoredDuringExecution:
#      - labelSelector:
#          matchExpressions:
#            - key: "app"
#              operator: In
#              values:
#              - obEnv.envid
#        topologyKey: "kubernetes.io/hostname"

```

Where:
- `replicaCount` is the number of Order Balancer instances to be created.
- `repository` is the repository name of the configured container registry.
- `tag` is the version name that is used when you create a final Order Balancer image from the container.
- `name` is the name of the imagepull-secret if it is configured. For more information about imagepull-secret, see "[Creating Secrets](https://docs.oracle.com/en/industries/communications/asap/8.0/cloud-native-deploy/creating-asap-cloud-native-instance.html#GUID-B968CC19-B6CA-44CF-B43D-132E692D23CF)".
- `envid` is the unique environment ID of the instance. This ID should include only the lower-case alphanumeric characters. For example, asapinstance1.
- `port` is the port number of the WebLogic Server where Order Balancer is deployed.
- `host` is the hostname of the podman container.
- `domain` is the Order Balancer WebLogic domain name as configured in `ob.properties` while creating image.
- `servicechannel.port` is the channel port when you create a channel in the WebLogic domain.
- `service.port` is the admin port of the WebLogic Server.
- `type` is the ingress controller type. This type can be TRAEFIK or GENERIC or OTHER.
- `enabled` is the status of the ingress controller whether it is enabled or not. The value is true or false. By default, this is set to true.
- `sslIncoming` is the status of the SSL/TLS configuration on incoming connections whether it is enabled or not. The configuration value is true or false. By default, this is set to false. If you want to set the value to true, create keys, certificate, and secret by following the instructions in the "[Setting Up ASAP Cloud Native for Incoming Access](https://docs.oracle.com/en/industries/communications/asap/8.0/cloud-native-deploy/integrating-asap.html#GUID-BB51B3EA-8683-469D-AA80-8DEE9A1B536F)" section.
- `adminsslhostname` is the host name of the https access.
- `adminhostname` is the host name of the http access.
- `secretName` is the secret name of the certificate created for SSL/TLS. For more information about creating keys and secret name, see "[Setting Up ASAP Cloud Native for Incoming Access](https://docs.oracle.com/en/industries/communications/asap/8.0/cloud-native-deploy/integrating-asap.html#GUID-BB51B3EA-8683-469D-AA80-8DEE9A1B536F)."
- `podAntiAffinity` can be uncommented to avoid co-locating two Order Balancer pods on the same node.
- `requiredDuringSchedulingIgnoredDuringExecution` tells the Kubernetes Scheduler that it should never co-locate two Pods which have app label as `envid` in the domain defined by the `topologyKey.`
- `topologyKey` with the value kubernetes.io/hostname indicates that the domain is an individual node.

Note: `adminsslhostname` and `secretName` are applicable only if `sslIncoming` is set to true.

2. Create PV and PVC for the Order Balancer instance. The pv path is used to store the wallet and logs of the ASAP instance added to the Order Balancer. The sample files are available in the `ob_cntk.zip` at `$OB_CNTK/samples/nfs/` file. Use the `hostPath` to access the specified path on the local nodes. Use `nfs` to access specified path on all the nodes.
   
Note: Creating PV and PVC is a mandatory step for the Order Balancer instance.
```yaml
# This is pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: <project>-<obEnv.envid>-nfs-pv
  labels:
    type: local
spec:
  storageClassName: wallet
  capacity:
    storage: 10Gi
  accessModes:    
    - ReadWriteMany  
# Valid values are Retain, Delete or Recycle  
persistentVolumeReclaimPolicy: Retain  
# hostPath:      
  nfs:
    server: <server>
    path: <path>
# This is pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: <obEnv.envid>-pvc
  namespace: <project>
spec:
  storageClassName: wallet
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Gi
```
Where:
- `<project>` is the namespace. In the above example, value is sr.
- `<obEnv.envid>` is the environment ID provided in the `$OB_CNTK/samples/charts/traefik/values.yaml` file. In the above example, value is ob96.

The below paths from both Order Balancer pods are mounted on to the PV path as:
For wallet files: `/u01/oracle/user_projects/domains/<domainName>/oracle_communications/asap` to `PV_mount_path/asap`

For log files of each Order Balancer pod: `/u01/oracle/user_projects/domains/<domainName>/oracle_communications/logs` to `PV_mount_path/OB_pod_name/logs`

For example, `/mnt/OB96-1/logs and /mnt/OB96-0/logs`

3. Run the following command to create pv and pvc: 
    `kubectl apply -f pv.yaml Kubectl apply -f pvc.yaml -n sr`
4. Verify that whether pv and pvc are created successfully or not by running the following command:
    `kubectl get pv kubectl get pvc -n sr`
5. Run the following command to create the Order Balancer instance:
	`$OB_CNTK/scripts/create-instance.sh -p sr -i quick`
The `create-instance.sh` script uses the Helm chart located in the `charts/ob` directory to deploy the Order Balancer image, service, and ingress controller for your instance. If the script fails, see "[Troubleshooting Issues with the Scripts]" before you make additional attempts.

6. Validate the important input details such as Image name and tag, specification files used (Values Applied), hostname, and port for ingress routing:
```bash

Calling helm lint
==> Linting ../charts/ob
[INFO] Chart.yaml: icon is recommended
 
1 chart(s) linted, 0 chart(s) failed
Project Namespace : sr
Instance Fullname : sr-quick
NAME              : sr-quick
LB_PORT           : 30305
LAST DEPLOYED     : Timestamp
NAMESPACE         : sr
STATUS            : deployed
REVISION          : 1
TEST SUITE        : n/a
```
If PV/PVC is not configured before, the Pod will be in the pending state as shown below:
```bash
kubectl describe pod <pod-name> -n sr
Events:
  Type     Reason            Age    From               Message
  ----     ------            ----   ----               -------
  Warning  FailedScheduling  2m31s  default-scheduler  persistentvolumeclaim "sr-pvc" not found
  Warning  FailedScheduling  2m31s  default-scheduler  persistentvolumeclaim "sr-pvc" not found
```

7. When the Pod state shows READY 1/1, your Order Balancer instance is up and running.

The base hostname required to access this instance using HTTP is `adminobhostnonssl.asap.org`. See "[[Planning and Validating Your Cloud Environment]]" for details about hostname resolution.

The `create-instance` script prints out the following valuable information that you can use when you work with your Order Balancer domain:

- The URL for accessing the WebLogic UI, which is provided through the ingress controller at host: `http://adminobhostnonssl.asap.org:30305/console`.

#### Validating the Order Balancer Instance
After creating an instance, you can validate it by accessing the WebLogic Server console.
Run the following command to display the Pod details of the Order Balancer instances that you have created:
```bash
$ kubectl get all -n OB_namespace
NAME                   READY        STATUS    RESTARTS   AGE
pod/ob96-0              1/1        Running        0      24h
pod/ob96-1              1/1        Running        0      24h
NAME                     TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)            AGE
service/ob96-hlservice   ClusterIP   None           <none>    6723/TCP,6724/TCP   3d18h
service/ob96-service     ClusterIP   10.106.240.147 <none>    6723/TCP,6724/TCP   3d18h
NAME                    READY   AGE
statefulset.apps/ob96   2/2     3d18h
```
NOTE: After the Order Balancer instances are created, it may take a few minutes to start Order Balancer servers.

To access WebLogic Remote Console outside the cluster, enter the following URL in the browser:

For non-SSL:
`http://adminhostnonssl.asap.org:nonSSL_Traefik_Port/console`

For SSL:
`https://adminhostssl.asap.org:SSL_Traefik_port/console`

The nonSSL_Traefik_Port and SSL_Traefik_port are mentioned in `$ASAP_CNTK/samples/charts/traefik/values.yaml`.

The system prompts for the user name and password. Enter the WebLogic domain user name and password.
Update the hosts file with the `hostname` and `master_ip` address on the machine where the URL is getting accessed.

Note: The hosts file is located in `/etc/hosts` on Linux and MacOS machines and in `C:\Windows\System32\drivers\etc\hosts` on Windows machines.
`ip_address adminhostnonssl.asap.org`

#### Scaling the Order Balancer Instance

You can create multiple instances of Order Balancer by setting the `replicaCount` accordingly in the `values.yaml` file. The Order Balancer helm chart templates create multiple instances as StatefulSet pods. The unique pod names are automatically configured by suffixing an ordinal number with the StatefulSet Name. For example, if the StatefulSet Name is OBapp, the pod names are OBapp-0, OBapp-1 and so on. The StatefulSet policy ensures that only one pod with the given name is run at a given point of time.

The number of Order Balancer instances can be scaled up or down by modifying the replicas count in StatefulSet:
`kubectl scale statefulsets stateful_set_name --replicas=replica_count`

Where
`stateful_set_name` is the name of the StatefulSet. In this example, it is OBapp.
`replica_count` is the number of the Order Balancer instances to be created.

