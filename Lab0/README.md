# Lab0 - Setup

## Pre-requisites:

1. An instance of IBM Cloud Kubernetes Service (IKS). Follow instructions here: https://digidevcon.gitbook.io/kubernetes-coderdojo/exercise-0a
2. An instance of IBM Cloud Container Registry, created when you created the first cluster on your account,

## Optional

To `Enable Issues` and `Track deployment of code changes` in the source provider, you need to have Admin permissions on your repository. You can create a fork of the IBM Guestbook application.

 * Go to https://github.com/IBM/guestbook,
 * Create a fork to your own Github repo,
 * [Optional] Clone the code to your localhost,

```console
$ git clone https://github.com:<username>/guestbook.git
Cloning into 'guestbook'...
```
    
## Installing IBM Cloud Developer Tools

You can use a web terminal with all required tools pre-installed, but you can also  install the [IBM Cloud Developer Tools CLI](https://cloud.ibm.com/docs/cli?topic=cloud-cli-getting-started) on your localhost, 

```console
$ curl -sL https://ibm.biz/idt-installer | bash
```

* This command will install the latest stand-alone `IBM Cloud CLI` plus the following tools:
    * Homebrew (Mac only),
    * Git,
    * Docker,
    * Helm,
    * kubectl,
    * curl,
    * IBM Cloud Developer Tools plug-in  (`ibmcloud dev`)
    * IBM Cloud Functions plug-in
    * IBM Cloud Object Storage plug-in
    * IBM Cloud Container Registry plug-in (`ibmcloud cr`)
    * IBM Cloud Kubernetes Service plug-in (`ibmcloud ks`)


## Access the IBM Cloud and IKS from your Client

The [AppModernization/Lab3](https://github.com/remkohdev/AppModernization/tree/master/Lab03) has instructions on how to connect to IBM Cloud and an IBM Kubernetes Service (IKS).

To login and configure your connection,
```
$ IBM_USERID=<your IBMCloud ID> IBM_PASSWORD='<IBMCloud Password>' IBM_CFAPI=https://api.us-south.cf.cloud.ibm.com IBM_ORG="<org name>" CLUSTER_NAME=<org name>iksuser<user number> ES_SVC_NAME=<org name>esuser<user number> REGION=us-south ZONE=dal10 SPACE=dev ACCOUNTID=<account id>

$ ibmcloud login -u $IBM_USERID -p $IBM_PASSWORD -c $ACCOUNTID -r $REGION -o "${IBM_ORG}" -s $SPACE
```

Configure the Resource Group for completion,
```
$ ibmcloud target -g default
```

Set the kubectl config context,
```
$ eval $(ibmcloud ks cluster config --cluster "${CLUSTER_NAME}" --export)
```

Test your connection,

```
$ kubectl version --short
$ kubectl config current-context
```