# ForgeRock Access Management + ForgeRock Directory Services on OpenShift

This examples assumes OpenShift Online 3 is used. 

Deployment can be rolled out using `openam-dj-template.yaml` Template. 
There's also `dj-statefulset.yaml` manifest which allows to deploy only Directory Services as a Stateful Set. Please bear in mind Stateful Set is still in technology preview stage on OpenShift Online.


### Requirements
* OpenShift Online account
* Most likely Pro subscription as multiple pods of certail memory/cpu/storage requirements need to be deployed. 
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
$ oc process --parameters -f openam-dj-template.yaml
```
Values in `Values` column are default when parameter is not specified. In our example the only required parameter is `IMAGEREPO`.

###### Result
The output should be similar to this:

```sh
NAME                DESCRIPTION                                                        GENERATOR           VALUE
DEPLOYMENT          Deployment name                                                                        openam
IMAGEREPO           Pointer to Docker Image repository eg. docker.io/user/repository
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

config.json might use `credsStore` like `osxkeychain` in Mac OS X. In this case you can either disable "Securely store docker logins in macOS keychain" temporarily and reenable it right after performing the step below. You can also create that file using the above as template and substituting value of `auth` key with Base64 encoded `dockerid:password` string. 

You need to Base64 encode the entire file:
```sh
cat ~/.docker/config.json | base64
```

This would give Base64 encoded output similar to this:
`ewogICJhdXRocyIgOiB7CiAgICAiaHR0cHM6Ly9pbmRleC5kb2NrZXIuaW8vdjEvIiA6IHsKICAgICAgImF1dGgiIDogInNkZmx3a2VyamwyazNqNGwyazM0VSIKICAgIH0KICB9Cn0K`

Use that string `DOCKER_CREDS` parameter in the following points. 
You can also export it as ENV variable to make life easier. DO NOT forget to clean up as this exposes your Docker credentials.

```sh
$ export DOCKER_CREDS=`cat ~/.docker/config.json | base64`
```

##### Run using parameters

```sh
$ oc process -f openam-dj-template.yaml -p DEPLOYMENT=demo -p DOCKER_CREDS=$DOCKER_CREDS -p IMAGEREPO=docker.io/mydockerid/openam-eval | oc create -f -
```

##### Check deployment status

```sh
$ oc get all
```

After all has been successfully created you should see output similar to this:
```sh
NAME                  DOCKER REPO                     TAGS           UPDATED
imagestreams/demo     docker.io/yourdockerid/openam-eval   latest,5.5.1   2 hours ago
imagestreams/opendj   docker.io/yourdockerid/opendj         latest         2 hours ago

NAME                     REVISION   DESIRED   CURRENT   TRIGGERED BY
deploymentconfigs/demo   1          1         1         config,image(demo:latest)

NAME          HOST/PORT                                        PATH      SERVICES   PORT       TERMINATION   WILDCARD
routes/demo   demo-test.e4ff.pro-eu-west-1.openshiftapps.com   /openam   demo       8080-tcp                 None

NAME                  DESIRED   CURRENT   AGE
statefulsets/opendj   1         1         1h

NAME                     REVISION   DESIRED   CURRENT   TRIGGERED BY
deploymentconfigs/demo   1          1         1         config,image(demo:latest)

NAME          HOST/PORT                                        PATH      SERVICES   PORT       TERMINATION   WILDCARD
routes/demo   demo-test.e4ff.pro-eu-west-1.openshiftapps.com   /openam   demo       8080-tcp                 None

NAME              READY     STATUS    RESTARTS   AGE
po/demo-1-nrtt7   1/1       Running   0          2h
po/opendj-0       1/1       Running   0          1h

NAME        DESIRED   CURRENT   READY     AGE
rc/demo-1   1         1         1         2h
```


##### Delete what you deployed
```sh
$ oc process -f openam-dj-template.yaml -p DEPLOYMENT=demo -p IMAGEREPO=docker.io/mydockerid/myrepo | oc delete -f -
```

#### Deploy Directory Services as Stateful Set only
To deploy/delete Directory Services as Stateful Set you can proceed as follows:
Use `dj-statefulset.yaml` manifest instead. It doesn't take any parameters deploying DS with:
- with initial and maximum memory size of 3Gi
- CPU = 1

NOTE: it requires Volume called "opendj" presence.

To create:

```sh
$ oc deploy -f dj-statefulset.yaml
```

To remove:
```sh
$ oc delete -f dj-statefulset.yaml
```