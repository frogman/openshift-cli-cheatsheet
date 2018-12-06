# openshift-cli-cheatsheet
*For Quick Access*

http://bit.ly/gineesh | www.techbeatly.com
Refer forked repo : https://github.com/ginigangadharan/openshift-cheatsheet
https://gist.github.com/rafaeltuelho/111850b0db31106a4d12a186e1fbc53e

## OpenShift  CLI
oc command line tool will be installed on all master and node machines during cluster installation. You can also install oc utility on any other machines which is not part of openshift cluster. 
Download oc cli tool from : https://www.okd.io/download.html

On a RHEL system with valid subscription you can install with yum as below.
```
$ sudo yum install -y atomic-openshift-clients
```

Many common oc operations are invoked using the following syntax:
```
$ oc <action> <object_type> <object_name_or_id>
```

## Login and Logout
```
oc login https://10.142.0.2:8443 -u admin -p openshift 
                              # Login to openshift cluster
oc whoami                     # identify the current login
oc login -u system:admin      # login to cluster from any master node without a password
oc logout                     # logout from cluster
```
## oc status
```
oc status -v                  # get oc cluster status
```
## Managing Projects
```
oc get projects               # list Existing Projects
oc get project                # Display current project
oc project myproject          # switch to a project
oc new-project testlab --display-name='testlab' --description='testlab'        
                              # create a new project
oc adm new-project testlab --node-selector='project101=testlab'
                              # create a new project with node-selector. 
                              # Project pods will be created only those nodes with a label "project101=testlab"
oc delete project testlab     # delete a project
oc delete all --all           # delete all from a project
```
## Resources
```
oc get all                    # list all resource items
                                -w  watches the result output in realtime.
oc get nodes                  # list nodes in a cluster
oc get node/NODE_NAME -o yaml
                              # to see a node’s current capacity and allocatable resources
oc get nodes --show-labels | grep -i "project101=testlab"
                              # show nodes info with lable and list only node with a lable "project101=testlab"
```
## Managing Users
```
oc adm policy add-cluster-role-to-user cluster-admin develoer
```

## oc describe 
```
oc describe node <node1>      # show deatils of a specific resource
oc describe policybindings :default -n <project/namsspace>
                              # OCP 3.7 < show details of a project policy details 
oc describe rolebinding.rbac -n PROJECT_NAME
                              # OCP 3.7 > show details of a project policy details 
```
## oc Export 
```
oc export pod mysql-1-p1d35   
                              # export a definition of a resource (creating a backup etc) in JSON or YAML format.
```
## Managing pods
```
oc get pods                   # list running pods inside a project
oc get pods -o wide           # detailed listing of pods
oc get pod -o name            # for pod names
oc get pods -n *<project>*    # list running pods inside a project/name-space
oc get po POD_NAME -o=jsonpath="{..image}"
                              # get othe pod image details
oc get po POD_NAME -o=jsonpath="{..uid}"
                              # get othe pod uid details
oc adm manage-node NODE_NAME --list-pods
                              # list all pods running on specific node
```
## PVC - *PhysicalVolumeClaim* 
``` 
oc get pvc                    # list pvc
```
## oc exec - execute command inside a containe
```
oc exec  <pd> -i -t -- <command> 
                              # run command inside a container without login
                                eg: oc exec  my-php-app-1mmh1 -i -t -- curl -v http://dbserver:8076
```
## Events and Troubleshooting
```
oc get events                 # list events inside cluster
oc logs POD                   # get logs from pod
oc logs <pod> --timestamps    
oc logs -f bc/myappx          
oc rsh <pod>                  # login to a pod
```

## Understand and Help
```
oc explain <resource>         # documentation of a resource and its fields
                                eg: oc explain pod
                                    oc explain pod.spec.volumes.configMap
```
## Applications
```
oc new-app mysql MYSQL_USER=user MYSQL_PASSWORD=pass MYSQL_DATABASE=mydb -l db=mysql
                              # create a new application
oc new-app --docker-image=myregistry.example.com/dockers/myapp --name=myapp
                              # create a new application from private registry
oc new-app https://github.com/techbeatly/python-hello-world --name=python-hello
                              # create a new application from source code (s2i)
```
## Get Help
```
# oc help                     # list oc command help options
```
## Build from image
```
oc new-build openshift/nodejs-010-centos7~https://github.com/openshift/nodejs-ex.git --name='newbuildtest'
```

## Enable/Disable scheduling
```
oadm manage-node mycbjnode --schedulable=false 
                              # Disable scheduling on node
```

## Resource quotas

Hard constraints how much memory/CPU your project can consume

```
oc create -f <YAML_FILE_with_kind: ResourceQuota> -n PROJECT_NAME
                              # create quota details with YAML tempalte where kind should ResourceQuota
                              # Sample : https://github.com/ginigangadharan/openshift-cli-cheatsheet/blob/master/quota-template-32Gi_no_limit.yaml
oc describe quota -n PROJECT_NAME
                              # describe the quota details
oc get quota -n PROJECT_NAME  
                              # get quota details of the project
oc delete quota -n PROJECT_NAME 
                              # delete a quota for the project
```
## Labels & Annotations
1. Label examples: release, environment, relationship, dmzbased, tier, node type, user type
    - Identifying metadata consisting of key/value pairs attached to resources
2. Annotation examples: example.com/skipValidation=true, example.com/MD5checksum-1234ABC, example.com/BUILDDATE=20171217
    - Primarily concerned with attaching non-identifying information, which is used by other clients such as tools or libraries

```
oc patch node NODE_NAME -p '{"metadata": {"labels": {"project101":"testlab"}}}'
                              # add label
oc label secret ssl-secret env=test
                              # add label                              
```
## Limit ranges

- mechanism for specifying default project CPU and memory limits and requests

```
oc get limits -n development

oc describe limits core-resource-limits -n development
```
## ClusterQuota or ClusterResourceQuota
Ref: https://docs.openshift.com/container-platform/3.3/admin_guide/multiproject_quota.html
```
oc create clusterquota for-user-developer --project-annotation-selector openshift.io/requester=developer --hard pods=8
oc get clusterresourcequota |grep USER
                              # find the clusterresourcequota for USER
oc describe clusterresourcequota USER
```
## Config View
```
oc config view                  # command to view your current, full CLI configuration
                                  also can see the cluster url, project url etc.
```
## Managing Environment Variables
https://docs.openshift.com/enterprise/3.0/dev_guide/environment_variables.html
```
oc env rc/RC_NAME --list -n PROJECT
                                # list environment variable for the rc
oc env rc my-newapp MAX_HEAP_SIZE=128M
                                # set environment variable for the rc
```
## Security Context Constraints
```
oc get scc
```


## The replication controller
<to be done>

oc describe RESOURCE RESOURCE_NAME

oc export

oc create

oc edit

oc exec POD_NAME <options> <command>

oc rsh POD_NAME <options>

oc delete RESOURCE_TYPE name

oc version

docker version

oc cluster up \
  --host-data-dir=... \
  --host-config-dir=...

oc cluster down

oc cluster up \
  --host-data-dir=... \
  --host-config-dir=... \
  --use-existing-config

```

```


oc project myproject

oc expose service cotd
```

## Create persistent volume

- Supports stateful applications

- Volumes backed by shared storage which are mounted into running pods

- iSCSI, AWS EBS, NFS etc.

## Create volume claim

- Manifests that pods use to retreive and mount the volume into pod at initialization time

- Access modes: REadWriteOnce, REadOnlyMany, ReadWriteMany

## Deployments


### Deployment strategies

#### Rolling

#### Triggers

#### Recreate

#### Custom

#### Lifecycle hooks

#### Deployment Pod Resources

### Blue-Green deployments

```
oc new-app https://github.com/devops-with-openshift/bluegreen#green --name=green

oc patch route/bluegreen -p '{"spec":{"to":{"name":"green"}}}'

oc patch route/bluegreen -p '{"spec":{"to":{"name":"blue"}}}'
```

### A/B Deployments

```
oc annotate route/ab haproxy.router.openshift.io/balance=roundrobin

oc set route-backends ab cats=100 city=0

oc set route-backends ab --adjust city=+10%
```

### Canary Deployments

### Rollbacks

```
oc rollback cotd --to-version=1 --dry-run

oc rollback cotd --to-version=1

oc describe dc cotd
```


## Pipelines




oc new-app jenkins-pipeline-example

oc start-build sample-pipeline


```

- Customizing Jenkins:

```
vim openshift.local.config/master/master-confi.yaml

jenkinsPipelineConfig:
  autoProvisionEnabled: true
  parameters:
    JENKINS_IMAGE_STREAM_TAG: jenkins-2-rhel7:latest
    ENABLE_OAUTH: true
  serviceName: jenkins
  templateName: jenkins-ephemeral
  templateNamespace: openshift
  ```
  
  - Good resource for Jenkinsfiles: https://github.com/fabric8io/fabric8-jenkinsfile-library
  
## Configuration Management

## Secrets

### Creation

- /!\ Maximum size 1MB /!\

```
oc secret new test-secret cert.pem

oc secret new ssl-secret keys=key.pem certs=cert.pem

oc get secrets --show-labels=true

oc delete secret ssl-secret
```

### Using secrets in Pods

- Mounting the secret as a volume

```
oc volume dc/nodejs-ex --add -t secret --secret-name=ssl-secret -m /etc/keys --name=ssl-keys deploymentconfigs/nodejs-ex

oc rsh nodejs-ex-22-8noey ls /etc/keys
```

- Injecting the secret as an env var

```
oc secret new env-secrets username=user-file password=password-file

oc set env dc/nodejs-ex --from=secret/env-secret

oc env dc/nodejs-ex --list
```

## Configuration Maps

- Similar to secrets, but with non-sensitive text-based configuration

### Creation

```
oc create configmap test-config --from-literal=key1=config1 --from-literal=key2=config2 --from-file=filters.properties

oc volume dc/nodejs-ex --add -t configmap -m /etc/config --name=app-config --configmap-name=test-config
```

### Reading config maps

```
oc rsh nodejs-ex-26-44kdm ls /etc/config
```

### Dynamically change the config map

```
oc delete configmap test-config

<CREATE AGAIN WITH NEW VALUES>

<NO NEED FOR MOUNTING AS VOLUME AGAIN>
```

### Mounting config map as ENV

```
oc set env dc/nodejs-ex --from=configmap/test-config

oc describe pod nodejs-ex-27-mqurr
```

## ENV

### Adding

```
oc set env dc/nodejs-ex ENV=TEST DB_ENV=TEST1 AUTO_COMMIT=true

oc set env dc/nodejs-ex --list
```

### Removing

```
oc set env dc/nodejs-ex DB_ENV-
```

## Change triggers

1. `ImageChange` - when uderlying image stream changes

2. `ConfigChange` - when the config of the pod template changes



## OpenShift Builds

### Build strategies

- Source-to-Image (S2I): uses the opensource S2I tool to enable developers to reporducibly build images by layering the application's soure onto a container image

- Docker: using the Dockerfile

- Pipeline: uses Jenkins, developers provide Jenkinsfile containing the requisite build commands

- Custom: allows the developer to provide a customized builder image to build runtime image

### Build sources

- Git

- Dockerfile

- Image

- Binary

### Build Configurations

- contains the details of the chosen build strategy as well as the source

```
oc new-app https://github.com/openshift/nodejs-ex

oc get bc/nodejs-ex -o yaml
```

- unless specified otherwise, the `oc new-app` command will scan the supplied Git repo. If it finds a Dockerfile, the Docker build strategy will be used; otherwise source strategy will be used and an S2I builder will be configured


### S2I

- Components:

1. Builder image - installation and runtime dependencies for the app

2. S2I script - assemble/run/usage/save-artifacts/test/run

- Process:

1. Start an instance of the builder image

2. Retreive the source artifacts from the specified repository

3. Place the source artifacts as well as the S2I scripts into the builder image (bundle into .tar and stream into builder image)

4. Execute assemble script

5. Commit the image and push to OCP registry

- Customize the build process:

1. Custom S2I scripts - their own assemble/run etc. by placing scripts in .s2i/bin at the base of the source code, can also contain environment file

2. Custom S2I builder - write your own custom builder

#### Adding a New Builder Image

#### Building a Sample Application

#### Troubleshooting

- Adding the --follow flag to the start-build command

- oc get builds

- oc logs build/test-app-3

- oc set env bc/test-app BUILD_LOGLEVEL=5 S2I_DEBUG=true

## Application Management

- Operational layers:

1. Operating system infrastructure operations - compute, network, storage, OS

2. Cluster operations - cluster managemebt OpenShift/Kubernetes

3. Application operations - deployments, telemetry, logging

### Integrated logging

- the EFK (Elasticsearch/Fluentd/Kibana) stack aggregates logs from nodes and application pods

```
oc cluster up --logging=true
```

### Simple metrics

- the Kubelet/Heapster/Cassandra and you can use Grafana to build dashboard

```
oc cluster up --metrics=true
```

### Resource scheduling

- default behavior:

1. best effor isolation = no primises what resources can be allocated for your project

2. might get defaulted values

3. out of memory killed randomly

4. might get CPU starved (wait to schedule your workload)



### Multiproject quota

- you may use project labels or annotations when creating multiproject spanning quotas

```
oc login -u system:admin


oc login -u developer -p developer

oc describe AppliedClusterResourceQuota
```

### Auto scaling of the pod

```
oc autoscale dc myapp --min 1 --max 4 --cpu-percent=75

oc get hpa myapp
```
