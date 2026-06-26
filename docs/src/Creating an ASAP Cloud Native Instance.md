# Creating an ASAP Cloud Native Instance

This chapter describes how to create an ASAP cloud native instance in your cloud environment using the operational scripts and the base ASAP configuration provided in the ASAP cloud native toolkit. You can create an ASAP instance quickly to become familiar with the process, explore the configuration, and structure your own project. This procedure is intended to validate that you are able to create an ASAP instance in your environment.

Before you create an ASAP instance, you must do the following:

- Download the ASAP cloud native tar file and extract the `asap-cntk.zip` file. For more information about downloading the ASAP cloud native toolkit, see "[[Downloading the ASAP Cloud Native Artifacts]]".
- Install the Traefik container images

### Installing the ASAP Cloud Native Artifacts and the Toolkit

Build container image for the ASAP application using the ASAP cloud native Image Builder.
You must create a private repository for this image, ensuring that all nodes in the cluster have access to the repository.

Copy the ASAP cloud native toolkit that is the `asap-cntk.zip` file to one of the nodes in the Kubernetes cluster:

- On Oracle Linux: Where Kubernetes is hosted on Oracle Linux, download and extract the tar archive to each host that has connectivity to the Kubernetes cluster.

Set the variable for the installation directory by running the following command, where asap_cntk_path is the installation directory of the ASAP cloud native toolkit:
**`$ export ASAP_CNTK=asap_cntk_path`**

### Installing the Traefik Container Image
~~NOTE: If you are installing Order Balancer in the ASAP namespace, ignore this section.~~

To leverage the ASAP cloud native samples that integrate with Traefik, the Kubernetes environment must have the Traefik ingress controller installed and configured.

If you are working in an environment where the Kubernetes cluster is shared, confirm whether Traefik has already been installed and configured for ASAP cloud native. If Traefik is already installed and configured, set your `TRAEFIK_NS` environment variable to the appropriate name space.

Check that any roles and rolebindings created by Traefik are removed. There could be a clusterrole and clusterrolebinding called "traefik-operator". There could also be a role and rolebinding called "traefik-operator" in the $TRAEFIK_NS name space. Delete all of these before you set up Traefik.

```bash
kubectl get clusterrole -n $TRAEFIK_NS 
kubectl get clusterrolebinding -n $TRAEFIK_NS 
kubectl delete clusterrole <clusterrole_name> -n $TRAEFIK_NS 
kubectl delete clusterrolebinding <clusterrolebinding_name> -n $TRAEFIK_NS 
```

To download and install the Traefik container image:

1. Ensure that Podman in your Kubernetes cluster can pull images from the container registry.
2. Run the following command to create a name space ensuring that it does not already exist:
   NOTE: You might want to add the traefik name space to the environment setup such as `.bashrc`.
```bash
kubectl get namespaces 
export TRAEFIK_NS=traefik
kubectl create namespace $TRAEFIK_NS
```
3. Run the following commands to install Traefik using the `$ASAP_CNTK/samples/charts/traefik/values.yaml` file in the samples:
```bash
helm repo add traefik https://traefik.github.io/charts
helm install traefik-operator traefik/traefik \
 --namespace $TRAEFIK_NS \
 --values $ASAP_CNTK/samples/charts/traefik/values.yaml \
  --set "kubernetes.namespaces={$TRAEFIK_NS}" 
   --version 26.0.0
```
where repo_link is [https://helm.traefik.io/traefik](https://helm.traefik.io/traefik) or [https://traefik.github.io/charts](https://traefik.github.io/charts) based on the helm chart version. For more details, see: [https://github.com/traefik/traefik-helm-chart](https://github.com/traefik/traefik-helm-chart)

After the installation, Traefik monitors the name spaces listed in its `kubernetes.namespaces` field for Ingress objects.

When the `values.yaml` Traefik sample in the ASAP cloud native toolkit is used as is, Traefik is exposed to the network outside of the Kubernetes cluster through port 30305. To use a different port, edit the YAML file before installing Traefik. Traefik metrics are also available for Prometheus to scrape from the standard annotations.

**Traefik function can be viewed using the Traefik dashboard.** Create the Traefik dashboard by running the instructions provided in the `$ASAP_CNTK/samples/charts/traefik/traefik-dashboard.yaml` file. **To access this dashboard, the URL is: `http://traefik.asap.org`. This is if you use the `values.yaml` file provided with the ASAP cloud native toolkit; it is possible to change the hostname as well as the port to your desired values.**

### Creating an ASAP Instance

This section describes how to create an ASAP instance.
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
`$ASAP_CNTK/scripts/register-namespace.sh -p sr -t targets # For example, $ASAP_CNTK/scripts/register-namespace.sh -p sr -t traefik`

#### Creating an ASAP Instance

This procedure describes how to create an ASAP instance in your environment using the scripts that are provided with the toolkit.

To create an ASAP instance:

1. Update the `$ASAP_CNTK/charts/asap/values.yaml` file with the following values:
```yaml
# Default values for asap.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

replicaCount: 1
image:
  repository: asapcn
  pullPolicy: IfNotPresent
  tag: "8.0.0.0.0"
imagePullSecrets:
  - name: asap-imagepull
asapEnv:
  envid: cne1
  port: 7890
  host: asaphost
persistence:
  enabled: false
readiness:
  enabled: true
  initialDelaySeconds: 240
  periodSeconds: 60
liveness:
  enabled: true
  periodSeconds: 120
  initialDelaySeconds: 120
  failureThreshold: 3
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
  port: 7891
service:
  name: adminport
  type: ClusterIP
  port: 7890

ingress:
  type: TRAEFIK
  enabled: true
  sslIncoming: false
  adminsslhostname: adminhost.asap.org
  adminhostname: adminhostnonssl.asap.org
  secretName: project-instance-asapcn-tls-cert
  hosts:
    - host: adminhost.asap.org
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

affinity: {}

```

The variables in the example have the following values:
- `repository` is the repository name of the configured container registry
- `tag` is the version name that is used when you create a final ASAP image from the container
- `name` is the name of the imagepull-secret if it is configured. For more information about imagepull-secret, see "[[Creating Secrets]]".
- `envid` is the unique environment ID of the instance. This ID should include only the lower-case alphanumeric characters. For example, asapinstance1.
- `port` is the port number of the WebLogic Server where ASAP is deployed.
- `host` is the hostname of the Podman container.
**Note**
    **The hostname should match with the hostname when you create the ASAP image. If the hostname mismatches, the ASAP servers may not start.**
- `persistence.enabled` is set to true if PVC is enabled.
- `readiness.enabled` is set to true to enable the readiness probe in Kubernetes. You configure the readiness and liveness probes in Kubernetes to restart the failed ASAP instances automatically and to restore the services. Kubernetes uses the liveness and readiness probes to find the failed ASAP instances and restarts those instances to restore the services automatically. Kubernetes uses the readiness probe to know when a container is ready to start accepting traffic.
- `initialDelaySeconds` is the number of seconds after the ASAP instance has started before the readiness probes are initiated.
- `periodSeconds` specifies how often (in seconds) to perform the readiness probe.
- `liveness.enabled` is set to true to enable the liveness probe. Kubernetes uses the liveness probes to know when to restart a container.
- `periodSeconds` specifies how often (in seconds) to perform the liveness probe.
- `initialDelaySeconds` is the number of seconds after the ASAP instance has started before the liveness probes are initiated.
- `failureThreshold` is the number of times that Kubernetes retry the liveness probes before restarting the ASAP instance.
- `servicechannel.port` is the channel port when you create a channel in the WebLogic domain.
- `service.port` is the admin port of the WebLogic Server.
- `type` is the ingress controller type. This type can be TRAEFIK or GENERIC or OTHER.
- `enabled` is the status of the ingress controller whether it is enabled or not. The value is true or false. By default, this is set to true.
- `sslIncoming` is the status of the SSL/TLS configuration on incoming connections whether it is enabled or not. The configuration value is true or false. By default, this is set to false. If you want to set the value to true, create keys, certificate, and secret by following the instructions in the "[Setting Up ASAP Cloud Native for Incoming Access](https://docs.oracle.com/en/industries/communications/asap/8.0/cloud-native-deploy/integrating-asap.html#GUID-BB51B3EA-8683-469D-AA80-8DEE9A1B536F)" section.
- `adminsslhostname` is the hostname of the https access.
- `adminhostname` is the hostname of the http access.
- `secretName` is the secret name of the certificate created for SSL/TLS. For more information about creating keys and secret name, see "[Setting Up ASAP Cloud Native for Incoming Access](https://docs.oracle.com/en/industries/communications/asap/8.0/cloud-native-deploy/integrating-asap.html#GUID-BB51B3EA-8683-469D-AA80-8DEE9A1B536F)."
**Note:**
	`adminsslhostname` and `secretName` are applicable only if `sslIncoming` is set to true.

2. Create PV and PVC if the persistence.enabled is set to true in the `$ASAP_CNTK/charts/asap/values.yaml` file for the ASAP instance. The PV path is used to store the logs of the ASAP instance. 
   
   The sample files are available in the `asap_cntk.zip` at `$ASAP_CNTK/samples/nfs/` file. Use the `hostPath` to access the local nodes. Use `nfs` to access all the nodes.
```yaml
# This is pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: project-asapEnv.envid-nfs-pv
  labels:
    type: local
spec:
  storageClassName: asaplogs
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
#  hostPath:
#   path: "/mnt/asap/logs/
  nfs:
    server: <server>
    path: <path>
# This is pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: <asapEnv.envid>-pvc
  namespace: sr
spec:
  storageClassName: asaplogs
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

Where:
- project is the namespace. In the above example, value is sr.
- asapEnv.envid is the environment ID provided in the `$ASAP_CNTK/charts/asap/values.yaml` file. In the above example, value is cne1.
This path will be mounted on the Pod as `/scratch/oracle/asap/DATA/logs/`

3. Run the following command to create the ASAP instance:
   `$ASAP_CNTK/scripts/create-instance.sh -p sr -i quick`
   **where `-p` is the project namespace and `-i` instance name**

   The `create-instance.sh` script uses the Helm chart located in the `charts/asap` directory to deploy the ASAP image, service, and ingress controller for your instance. If the script fails, see "[Troubleshooting Issues with the Scripts](https://docs.oracle.com/en/industries/communications/asap/8.0/cloud-native-deploy/creating-asap-cloud-native-instance.html#GUID-1470F248-1734-4C40-BB2E-F927C9E41F84)" before you make additional attempts.
4. After you run the `create-instance.sh` script, validate the important input details in the response such as Image name and tag, specification files used (Values Applied), hostname, and port for ingress routing:
```bash

1 chart(s) linted, 0 chart(s) failed
Project Namespace : sr
Instance Fullname : sr-quick

NAME: sr-quick
LAST DEPLOYED: Sun Feb 27 17:49:25 2022
NAMESPACE: sr
STATUS: deployed
REVISION: 1
TEST SUITE: None

```
5. If you query the status of the ASAP pod, the **READY** state of the ASAP pod displays **0/1** for several minutes when the ASAP application is starting.
   
    When the **READY** state shows **1/1,** your ASAP instance is up and running. You can then validate the instance by submitting work orders.
    The base hostname required to access this instance using HTTP is `quick.sr.asap.org`. See "[[Planning and Validating Your Cloud Environment]]" for details about hostname resolution.

The `create-instance` script prints out the following valuable information that you can use when you work with your ASAP domain:

- The T3 URL: `http://t3.quick.sr.asap.org` This is required for external client applications such as JMS and WLST.
- The URL for accessing the WebLogic UI, which is provided through the ingress controller at host: `http://admin.quick.sr.asap.org:30305/console`.