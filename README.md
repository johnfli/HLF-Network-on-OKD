# Deploy Hyper Ledger Fabric on Openshift Kubernetes Distribution

The open-source distributed ledger solution, [Hyper Ledger Fabric](https://github.com/hyperledger/fabric) (HLF), deployed on a kubernetes environment. [Openshift Kubernetes Distribution](https://github.com/openshift/okd) (OKD) is a community driven version of [Kubernetes](https://kubernetes.io/) bundled with built-in CI/CD and high-availability features, easily managed through a friendly web console.

## Repo Contents Description

```
.
└── k8s
    ├── bins [dir] -- directory containing all the necessary HLF binary scripts   
    ├── config [dir] -- directory containing all the necessary HLF config and crypto-config files
    ├── docker
    │   ├── acme-orderer
    │   |   └── Dockerfile -- to build acme-orderer docker image
    │   ├── acme-peer
    │   │   └── Dockerfile -- to build acme-peer docker image
    │   └── budget-peer
    │       └── Dockerfile -- to build budget-peer docker image
    ├── nodechaincode [dir]
    ├── okd
    |   ├── acme-orderer
    |   │   ├── acme-orderer.sfs.yaml -- to deploy a pod running the acme-orderer docker image
    |   │   └── acme-orderer.svc.yaml -- to deploy a service for the acme-orderer pod
    |   ├── acme-peer
    |   │   ├── acme-peer.sfs.yaml -- to deploy a pod running the acme-peer docker image
    |   │   └── acme-peer.svc.yaml -- to deploy a service for the acme-peer pod
    |   └── budget-peer
    |       ├── budget-peer.sfs.yaml -- to deploy a pod running the budget-peer docker image
    |       └── budget-peer.svc.yaml -- to deploy a service for the budget-peer pod
    ├── clean-all.sh -- to clean the config files
    └── init-setup.sh -- to create the files inside config directory
```

## Prerequisites 

### Clone the repo


1. Git clone the current repository to your local machine

   ```sh
   git clone https://github.com/johnfli/okd-hlf-network.git
   ```

### Generate the config & crypto-config files


### Create and push to Dockerhub Registry the HLF Images 


2. Login to dockerhub via docker terminal
	
	$ sudo docker login -u <dockerhub-username> -p '<dockerhub-password>'
	
	E.g.
	
	$ sudo docker login -u johnfli -p 'securepassword'

<details><summary><b>References</b></summary>

1. https://docs.docker.com/engine/reference/commandline/login/

</details>
	
3. Login to dockerhub on https://hub.docker.com/ and create the repositories needed to store the images that will be built for the various HLF elements

<details><summary><b>References</b></summary>

1. https://docs.docker.com/docker-hub/repos/

</details>

4. Navigate to the [docker](docker/) directory

   ```sh
   cd okd-hlf-network/docker
   ```

5. Issue the following commands to build the docker images:
	
    ```sh
    docker build -t <dockerhub-username>/<image-repository-name>:<tag-version> . -f <filepath-to-dockerfile>/Dockerfile
    ```	

    In our case, we will need to build three images:
	
    1. For the orderer of organization acme: k8s-hlf-acme-orderer
        ```sh
        docker build -t johnfli/k8s-hlf-acme-orderer:2.0 . -f docker/orderer/Dockerfile
        ```	
    2. For the peer of organization acme: k8s-hlf-acme-peer
          ```sh
          docker build -t johnfli/k8s-hlf-acme-peer:2.0 . -f docker/orderer/Dockerfile
          ```	

    3. For the peer of organization budget: k8s-hlf-budget-peer
        ```sh
        docker build -t johnfli/k8s-hlf-budget-peer:2.0 . -f docker/orderer/Dockerfile
        ```	

6.	*[Optional]* Validate that the images are created in the local repository of your machine by executing:
	
    ```sh
    docker images
    ```	
	
7. To push the images into the dockerhub repository execute:
    ```sh
    docker push johnfli/k8s-hlf-acme-orderer:1.0

    docker push johnfli/k8s-hlf-acme-peer:1.0

    docker push johnfli/k8s-hlf-budget-peer:1.0
    ```	


    <details><summary><b>References</b></summary>

    1. https://docs.docker.com/engine/reference/commandline/push/

    </details>

    <br>

    **NOTE**: Since 2017 there is a bug that remains open, and before pushing an image to dockerhub you have to logout and log back in through your terminal, otherwise the access will be denied:

    ```sh
    docker logout

    docker login # provide your creds in cli or on when prompted on terminal

    docker push johnfli/k8s-hlf-budget-peer:1.0
    ```	
    <details><summary><b>References</b></summary>

    1. https://stackoverflow.com/questions/41984399/denied-requested-access-to-the-resource-is-denied-docker

    </details>
	
8. *[Optional]* After pushing, validate that the images exist in the [repositories](https://hub.docker.com/repositories) you created on dockerhub.


## Deploy HLF components onto the HPE OKD Cluster

### Setup

1. First off, sign in to the HPE OKD Cluster [Web Console](https://oauth-openshift.apps.okd.seclab.local/oauth/authorize?client_id=console&redirect_uri=https%3A%2F%2Fconsole-openshift-console.apps.okd.seclab.local%2Fauth%2Fcallback&response_type=code&scope=user%3Afull&state=b3fa7ff0) using LDAP and providing the credentials `<username>_ext:<password><6-digit-google-auth-OTP`.

	E.g.
	```
	johnflionis_ext & securepassword123456
	```
2. On the administrator/developer drop-down menu on the top-left of the OKD Web Console, select the Administrator option.

### Deploy the StatefulSet Components

In order for the HLF network components to run on the kubernetes environment, we will need a pod running the equivalent container image, along-side with a persistent volume, in order for the ledger and other useful data to be preserved along any voluntary or involuntary pod restarts.

1. On the Project drop-down menu, select the project `gatekeeper-dev`,
2. Click on "Workloads", then select the type "StatefulSets", and click on the button `Create Statefulset` on the top-right corner of the screen
3. Copy the content of [acme-orderer.sfs.yaml](okd/acme-orderer/acme-orderer.sfs.yaml), paste it into the OKD editor, replacing the existing configuration, click the button `Create`
4. Wait for the pod creation, it should take about a couple of minutes before the circle turns blue and inside it states `1 Pod`,
5. The acme-orderer is ready
6. Repeat every step as is for the rest of the components, but for step 3, where you should copy & paste the corresponding YAML configuration, i.e.,
	- [acme-peer.sfs.yaml](okd/acme-peer/acme-peer.sfs.yaml)
	- [budget-peer.sfs.yaml](okd/budget-peer/budget-peer.sfs.yaml)


### Deploy the Service Components

In order for the kubernetes cluster resources to communicate across each other, each pod is assigned a dedicated service that preserves its name across pod restart and always resolves and load-balances to its associated pods.

1. Click on "Networking", then select the type "Services", and click on the button `Create Service` on the top-right corner of the screen
2. Copy the content of [acme-orderer.svc.yaml](okd/acme-orderer/acme-orderer.svc.yaml), paste it into the OKD editor, replacing the existing configuration, click the button `Create`
3. The acme-orderer service will be created instantly,
4. Repeat every step as is for the rest of the components, but for step 2, where you should copy & paste the corresponding YAML configuration, i.e.,
	- [acme-peer.svc.yaml](okd/acme-peer/acme-peer.svc.yaml)
	- [budget-peer.svc.yaml](okd/budget-peer/budget-peer.svc.yaml)

### Log into a Terminal and execute the HLF scripts

1. Click on "Workloads", then select the type "StatefulSets", on the navigation bar click on the tab `Pods`, click on the name of the pod, e.g. `acme-orderer-0`,
2. On the next screen, again on the navigation bar, select the last tab `Terminal`,
3. A terminal will appear
4. The directories `/var/hyperledger/config`, `/var/hyperledger/bins` contains the binaries, scripts, yamls and block files needed
5. The directory `/var/hyperledger/nodechaincode` contains the chaincode binary
6. The directory `/var/hyperledger/<comp>.<org>.com`, e.g. `/var/hyperledger/peer.acme.com`, contains the certificates, etc.

### Error encountered

However, we have run into a persmission denied, when attempting to create the channel genesis block from the [airlinechannel.tx](config/airlinechannel.tx) as presented below:

```sh
budget-peer-0:/var/hyperledger/bins$ ./submit-channel-create.sh 
2022-07-05 19:59:09.155 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2022-07-05 19:59:09.208 UTC [cli.common] readBlock -> INFO 002 Received block: 0
Error: open ../config/airlinechannel.block: permission denied
budget-peer-0:/var/hyperledger/bins$ 
```
Then, we observe that we get the same error for a simple file creation:

```sh
budget-peer-0:/var/hyperledger/bins$ touch ../config/file.txt
touch: ../config/file.txt: Permission denied
budget-peer-0:/var/hyperledger/bins$ 
```

This happens because OKD assigns custom users to pods, with non=root priviledges as a security precaution, whereas the files belong to the `root` user:

```sh
budget-peer-0:/var/hyperledger/bins$ whoami
1000610000
budget-peer-0:/var/hyperledger/bins$ ls -la ../config/
total 36
drwxr-xr-x    1 root     root            23 Jun 15 07:28 .
drwxr-xr-x    1 root     root            27 Jun 15 07:28 ..
-rwxrwxrwx    1 root     root          1578 May 25 08:21 airlinechannel.tx
-rwxrwxrwx    1 root     root           320 May 25 08:21 budget-peer-update.tx
-rwxrwxrwx    1 root     root         28532 Jul  8  2020 core.yaml
```

This a known issue to the Openshift community, as described in this [stackoverflow post](https://stackoverflow.com/questions/70109707/openshift-missing-permissions-to-create-a-file).

Remediation attempts:
- We have tried to remediate that with the workarround proposed in this [stack overflow post](https://stackoverflow.com/questions/42363105/permission-denied-mkdir-in-container-on-openshift), however the HPE OKD does not permit arbitrary user ID declarations in the YAML configuration.
	
- Also, we attempted loading the whole `/var/hyperledger` directory onto a persistent volume (PV), as we noticed that we could touch files on PV mountpaths, where our user seems to have complete ownership, e.g.
	```sh
	budget-peer-0:/var/hyperledger/bins$ touch ../../ledger/file.txt
	budget-peer-0:/var/hyperledger/bins$ ls -la ../../ledger/
	total 40
	drwxrwsr-x    8 root     10006100      4096 Jul  5 20:25 .
	drwxrwxr-x    1 root     root            20 Jul  4 15:38 ..
	drwxrwsr-x    2 10006100 10006100      4096 Jun 15 08:02 chaincodes
	drwxrws---    3 10006100 10006100      4096 Jun 15 08:02 externalbuilder
	-rw-r--r--    1 10006100 10006100         0 Jul  5 20:25 file.txt
	drwxrwsr-x   10 10006100 10006100      4096 Jun 15 08:02 ledgersData
	drwxrws---    3 10006100 10006100      4096 Jun 15 08:02 lifecycle
	drwxrws---    2 root     10006100     16384 Jun 15 08:02 lost+found
	drwxrwsr-x    2 10006100 10006100      4096 Jul  4 15:38 transientstore
	```
	However, this was unsuccessful as well.

- Finally, we attempted to change the ownership of the folders we are interested in, from inside the Dockerfiles, e.g. [acme-peer Dockerfile](docker/acme-peer/Dockerfile), as shown in the code below, which has been commented out, since it did not solve the problem either:
	```Dockerfile	
	#3. [PERMISSION DENIED REMEDIATION] Adding user permissions on specific directory 
	RUN chgrp -R 0 /var/hyperledger/config && \
		chmod -R g=u /var/hyperledger/config
	```
We will further elaborate on the errors encountered and provide update.
