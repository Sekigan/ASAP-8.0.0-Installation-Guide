
#### Deleting and Recreating Your Order Balancer Instance

Deleting Your Order Balancer Instance
To delete your Order Balancer instance, run the following command:
**`$OB_CNTK/scripts/delete-instance.sh -p sr -i quick`**

Re-creating Your Order Balancer Instance
When you delete an Order Balancer instance, the database state for that instance still remains unaffected. You can re-create multiple Order Balancer instance by using the same image.
To re-create an Order Balancer instance, run the following command:
**`$OB_CNTK/scripts/create-instance.sh -p sr -i quick`**

#### Cleaning Up the Environment
To clean up the environment:

1. Delete the instance:
    `$OB_CNTK/scripts/delete-instance.sh -p sr -i quick`
   
2. Delete the name space, which in turn deletes the Kubernetes name space and the secrets:
    `$OB_CNTK/scripts/unregister-namespace.sh -p sr -d -t target`
    
 Note: `traefik` is the name of the target for registration of the name space. The script uses TRAEFIK_NS to find this target. Do not provide the "traefik" target if you are not using Traefik.
    
3. Delete the Order Balancer database users.
4. Delete pv and pvc using the following commands: 
    `kubectl delete pv <pv-name> Kubectl delete pvc <pvc-name> -n sr`

#### Troubleshooting Issues with the Scripts
This section provides information about troubleshooting some issues that you may come across when running the scripts.

If you experience issues when running the scripts, check the "Status" section of the domain to see if there is useful information:
`kubectl describe pod name -n sr`

Cleanup Failed Instance
When a create-instance script fails, you must clean up the instance before making another attempt at instance creation.

Note: Do not retry running the `create-instance.sh` script or the `upgrade-instance.sh` script immediately to fix any errors, as they would return errors. The `upgrade-instance.sh` script may work but re-running it does not complete the operation.

To clean up the failed instance, delete the instance:
**`$OB_CNTK/scripts/delete-instance.sh -p sr -i quick`**

Recreating an Instance
If you encounter issues when creating an instance, do not try to re-run the `create-instance.sh` script as this will fail. Instead, perform the cleanup activities and then run the following command:
**`$OB_CNTK/scripts/create-instance.sh -p sr -i quick`**