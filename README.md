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

##### If you use private Docker image repo read this

You need to have access to Docker config file ie. `.dockerconfig` or `config.json` depending on the version.
It should have syntax similar to that:

```json
{
  "auths" : {
    "https://index.docker.io/v1/" : {
      "auth" : "sdflwkerjl2k3j4l2k34U"
    }
  }
}
```

You need to Base64 encode the entire file:
```sh
cat ~/.docker/config.json | base64
```

This would give Base64 encoded output similar to this:
`ewogICJhdXRocyIgOiB7CiAgICAiaHR0cHM6Ly9pbmRleC5kb2NrZXIuaW8vdjEvIiA6IHsKICAgICAgImF1dGgiIDogInNkZmx3a2VyamwyazNqNGwyazM0VSIKICAgIH0KICB9Cn0K`

Use that string `DOCKER_CREDS` parameter in the following points. 
You can also export it as ENV variable to make life easier

```sh
$ export DOCKER_CREDS=`cat ~/.docker/config.json | base64`
```

##### Run using parameters

```sh
$ oc process -f openam-template.yaml -p DEPLOYMENT=demo -p DOCKER_CREDS=$DOCKER_CREDS -p IMAGEREPO=docker.io/mydockerid/openam-eval | oc create -f -
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