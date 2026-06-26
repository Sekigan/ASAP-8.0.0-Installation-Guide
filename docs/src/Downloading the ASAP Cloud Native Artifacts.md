#### Downloading the ASAP Cloud Native Artifacts
To deploy ASAP and Order Balancer cloud native instances in the Kubernetes cluster:
1. Download the cloud native tar file, for example `ASAP.R8_0_0.Bversion.cn.tar` from **Oracle Software Delivery Cloud or from My Oracle Support Website**.    
    ###### Note : The cloud native tar file is present in the Linux platform only.
    Where x is the patch set version of the ASAP cloud native.
    version is the release version of the ASAP cloud native.
    
2. Copy the tar file to the machine where Podman and Kubernetes are installed.
   
3. Extract the tar file using the following command:
    `tar -xvf ASAP.R8_0_0.Bversion.cn.tar`
    The artifacts in the tar file are extracted to the same directory where you ran the command.
    
    The following zip files are extracted:
    - **asap-img-builder.zip**: To build ASAP image and Order Balancer image.
    - **asap-cntk.zip**: To create ASAP instance.
    - **ob-cntk.zip**: To create Order Balancer instance.