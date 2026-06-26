#### Creating Secrets
**You must store sensitive data and credential information in the form of Kubernetes Secrets** that the scripts and Helm charts in the toolkit consume. Managing secrets is out of the scope of the toolkit and must be implemented while adhering to your organization's corporate policies. Additionally, **ASAP cloud native does not establish password policies.**

For an ASAP could native instance, the following secrets are required:

- **imagepull-secret**: If the private registry or repository is password protected, create this secret.
- **tls-secret:** If the traefik ingress is ssl-enabled, create this secret. For more information about creating tls-secret, see "[Setting Up ASAP Cloud Native for Incoming Access](https://docs.oracle.com/en/industries/communications/asap/8.0/cloud-native-deploy/integrating-asap.html#GUID-BB51B3EA-8683-469D-AA80-8DEE9A1B536F)."
To create imagepull-secret:

1. Run the following command:
    `podman login`
    
2. Enter the credentials or access token.
    The login process creates or updates the `config.json` file that contains an authorization
    token.
    
3. Run the following command to run the secret:    
    `kubectl create secret generic asap-imagepull -n namespace`