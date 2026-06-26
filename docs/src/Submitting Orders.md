# Submitting Orders
After you install ASAP, install the POTS cartridge. You can submit ASAP orders over JMS and Web Services.

- To submit ASAP orders over JMS, use an external runJMSclient. The endpoint must be as follows:
  `System.${ENV_ID}.ApplicationType.ServiceActivation.Application.1-0;7-4;ASAP.Comp.MessageQueue`
  The connection factory's provider URL must be as follows:
  
  For non-SSL:
  `http://adminhostnonssl.asap.org:30305/`
  
  For SSL:
  `https://adminhostssl.asap.org:30443/`

- To submit ASAP orders over Web Services, type the following URL in the web browser and access the ASAP Web Services WSDL:
  
  `http://adminhostnonssl.asap.org:30305/env_id/Oracle/CGBU/Mslv/Asap/Ws?WSDL`
  
  HTTP protocol is used for a handshake with the application server to authenticate and request a web service client stub, which is used as the launch pad to talk to Web Services. Then the client can communicate with the ASAP Web Services using HTTP or HTTPS protocols.

### Accessing the OCA Client

Similar to the ASAP traditional deployment, ASAP cloud native also connects through the Order Control Application (OCA) client using thick client.

To configure the OCA client:

1. Add an entry to the `hosts` file to the machine (Linux or Windows) from where you are planning to launch the OCA client to resolve DNS. This entry contains the IP address of the master node and hostname of the ASAP instance running in a cloud native. You can obtain the hostname from the `values.yaml` file.
    
    The hosts configuration file is located at:
    
    - On Windows: `C:\Windows\System32\drivers\etc\hosts`
    - On Linux: `/etc/hosts`
    
    For example:  
    `<Kubernetes_Cluster_Master_IP> hostname provided in the values.yaml file`
    
2. Update the `TBL_SERVER_INFO` table in the CTRL database.
    The hostname column contains the hostname mentioned in the `values.yaml` file and `INFO` column contains `HTTPS:<ingressport>`.
    
    For example:
```
update tbl_server_info set HOSTNAME='adminhost.asap.org'; 
update tbl_server_info set INFO='HTTP:30305'; 
update tbl_server_info set INFO='HTTPS:30443';
```
    
Note : Based on the Ingressroute configuration use `HTTP` or `HTTPS`. The port number is the traefik nodeport.
    
3. Update the **oca.cfg** file. See, [Introduction to the Order Control Application](https://docs.oracle.com/pls/topic/lookup?ctx=en/industries/communications/asap/8.0/cloud-native-deploy&id=ASPOC-GUID-F0788F9F-BBD2-4994-B4A2-A026E01C9820) for more details.
4. Launch the thick client once the database is updated.
 Note: Repeat Steps 1 and 2, and create separate session blocks for each ASAP server to enable access to multiple ASAP environments from the same OCA. This allows you to identify which work order is processed by a specific ASAP instance.