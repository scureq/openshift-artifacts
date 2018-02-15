# ForgeRock on OpenShift

This examples assumes OpenShift Online 3 is used. 


### Requirements
* OpenShift Online account
* OpenShift command line client (from RedHat) installed called "oc". Use the latest version. You can get it from [OpenShift GitHub](https://github.com/openshift/origin/releases). 


### How to run

##### Login to OpenShift

##### Create OpenShift Project
For example `am-playground`

```sh
$ oc new-project am-playground
```

##### List available parameters
```sh
$ oc process --parameters -f openam-template.yaml
```
Values in `Values` column are default when parameter is not specified. In our example the only required parameter is `IMAGEREPO`.

###### Result

```sh
NAME                DESCRIPTION                                                        GENERATOR           VALUE
DEPLOYMENT          Deployment name                                                                        openam
IMAGEREPO           Pointer to Docker Image repository eg. docker.io/user/repository
MEMORY_INIT         Initial memory allocated to containers                                                 2Gi
MEMORY_LIMIT        Maximum amount of memory container can consume                                         2Gi
STORAGE             Persistant Volume Claim size                                                           2Gi
```

##### Run using parameters
```sh
$ oc process -f openam-template.yaml -p DEPLOYMENT=demo -p IMAGEREPO=docker.io/mydockerid/myrepo | oc create -f -
```

##### Check deployment status
```sh
$ oc get all
```

After all has been successfully created you should see output similar to this:
```sh
NAME                DOCKER REPO                     TAGS           UPDATED
imagestreams/demo   docker.io/mydockerid/openam-eval   latest,5.5.1   2 hours ago

NAME                     REVISION   DESIRED   CURRENT   TRIGGERED BY
deploymentconfigs/demo   1          1         1         config,image(demo:latest)

NAME          HOST/PORT                                           PATH      SERVICES   PORT       TERMINATION   WILDCARD
routes/demo   demo-am-test.sddd.pro-eu-west-1.openshiftapps.com   /openam   demo       8080-tcp                 None

NAME                     REVISION   DESIRED   CURRENT   TRIGGERED BY
deploymentconfigs/demo   1          1         1         config,image(demo:latest)

NAME                DOCKER REPO                     TAGS           UPDATED
imagestreams/demo   docker.io/mydockerid/openam-eval   5.5.1,latest   2 hours ago

NAME              READY     STATUS    RESTARTS   AGE
po/demo-1-55nr4   1/1       Running   0          2h

NAME        DESIRED   CURRENT   READY     AGE
rc/demo-1   1         1         1         2h
```


##### Delete what you deployed
```sh
$ oc process -f openam-template.yaml -p DEPLOYMENT=demo -p IMAGEREPO=docker.io/mydockerid/myrepo | oc delete -f -
```