###  Validating the ASAP Instance
After creating an instance, you can validate it by accessing the WebLogic Server console.

Run the following command to display the pod details of the ASAP instance that you have created:
```bash
$ kubectl get all -n sr
NAME                                   READY   STATUS    RESTARTS   AGE
pod/asapinstance1-deployment-9845fbcb6-8qq2h    1/1     Running   0          5d15h

NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
service/asapinstance1-service   ClusterIP   10.99.231.206   <none>        7890/TCP,7891/TCP   5d21h

NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/asapinstance1-deployment   1/1     1            1           5d21h

NAME                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/asapinstance1-deployment-d5bd787f8    0         0         0       5d15h
```

To get the ASAP server status enter into the pod by using the following command:
`kubectl exec -it asapinstance1-deployment-9845fbcb6-8qq2h bash -n sr`
`kubectl exec -it asapinstance1-deployment-ID bash -n sr`

You are now entered into the ASAP pod. Navigate to the ASAP installation directory by using the following command:
```bash
cd $ASAP_BASE 
source Environment_Profile 
status
```

The status of ASAP servers are displayed. Ensure that there are minimum 5 processes in the output of `status` command, one process each of Control server, Daemon server, SARM, NEP and JNEP. In ASAP 7.4.1, on successful startup, the minimum process count is 5 as compared to 8 processes 
in 7.3.0 and 7.4.0.For example:
```bash
[root@asaphost asap]# status
               **** ASAP Application Status ****

    #  CPU       PID      Program                                  Application Location
    -- --------- -------- ---------------------------------------- ----------- --------
    1  00:00:16  251011   $ASAP_BASE/programs/ctrl_svr             CTRLCNE1    LOCAL   
    2  00:00:30  251381   $ASAP_BASE/programs/sarm                 SARMCNE1    LOCAL   
    3  00:00:08  251497   java                                     DAEMCNE1    LOCAL   
    4  00:00:08  279257   java                                     JNEP_CNE1   LOCAL   
    5  00:00:07  279319   $ASAP_BASE/programs/asc_nep              NEP_CNE1    LOCAL   

               **** End of Application Status ****
```

`CNE1` is the ENV_ID.

NOTE: After an ASAP instance is created, it may take a few minutes to start ASAP servers and WebLogic Server.

To access WebLogic Remote Console outside the cluster, enter the following URL in the browser:
`http://adminhostnonssl.asap.org:30305/console`

The system prompts for the user name and password. Enter the WebLogic domain user name and password.

Update the hosts file with the hostname and master_ip address on the machine where the URL is getting accessed with the following information:
`ip_address adminhostnonssl.asap.org`

NOTE: The hosts file is located in `/etc/hosts` on Linux and MacOS machines and in `C:\Windows\System32\drivers\etc\hosts` on Windows machines.

### Deleting and Recreating Your ASAP Instance

To delete your ASAP instance, run the following command:

`$ASAP_CNTK/scripts/delete-instance.sh -p sr -i quick`

Recreating Your ASAP Instance

When you delete an ASAP instance, the database state for that instance still remains unaffected. You can re-create an ASAP instance by using the same ASAP image.

To re-create an ASAP instance, run the following command:

`$ASAP_CNTK/scripts/create-instance.sh -p sr -i quick`

- Note: After recreating an instance, client applications such as SoapUI may need to be restarted to avoid using expired cache information.

If another ASAP instance is created in the same database using the same Environment ID, the ASAP installer deletes the previous ASAP database users and recreates new users.

**You should not create multiple ASAP instances with the same ASAP image.**

### Cleaning Up the Environment

To clean up the environment:

1. Delete the instance:
    `$ASAP_CNTK/scripts/delete-instance.sh -p sr -i quick`
    
2. Delete the name space, which in turn deletes the Kubernetes name space and the secrets:
    
    `$ASAP_CNTK/scripts/unregister-namespace.sh -p sr -d -t target`
    
- Note:  `traefik` is the name of the target for registration of the name space. The script uses TRAEFIK_NS to find this target. Do not provide the "traefik" target if you are not using Traefik.
    
1. Delete the ASAP database users.

### Troubleshooting Issues with the Scripts

This section provides information about troubleshooting some issues that you may encounter when running the scripts.
If you experience issues when running the scripts, do the following:

- Check the "Status" section of the domain to see if there is useful information:  
    `kubectl describe pod name -n sr`

Cleanup Failed Instance

When a create-instance script fails, you must clean up the instance before making another attempt at instance creation.

Note: Do not retry running the `create-instance` script or the `upgrade-instance` script immediately to fix any errors, as they would return errors. The `upgrade-instance` script may work but re-running it does not complete the operation.

To clean up the failed instance:

1. Delete the instance:
    `$ASAP_CNTK/scripts/delete-instance.sh -p sr -i quick`
    
Recreating an Instance
If you encounter issues when creating an instance, do not try to re-run the `create-instance.sh` script as this will fail. Instead, perform the cleanup activities and then run the following command:

`$ASAP_CNTK/scripts/create-instance.sh -p sr -i quick`