# Helm Chart for deployment of WSO2 Identity Server 

## Contents

* [Prerequisites](#prerequisites)
* [Quick Start Guide](#quick-start-guide)

## Prerequisites

* In order to use WSO2 Helm resources, you need an active WSO2 subscription. If you do not possess an active WSO2
  subscription already, you can sign up for a WSO2 Free Trial Subscription from [here](https://wso2.com/free-trial-subscription).<br><br>

* Install [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git), [Helm](https://github.com/kubernetes/helm/blob/master/docs/install.md)
(and Tiller) and [Kubernetes client](https://kubernetes.io/docs/tasks/tools/install-kubectl/) (compatible with v1.10) in order to run the 
steps provided in the following quick start guide.<br><br>

* An already setup [Kubernetes cluster](https://kubernetes.io/docs/setup/pick-right-solution/).<br><br>

* Install [NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/deploy/). This can
 be easily done via 
  ```
  helm install stable/nginx-ingress --name nginx-wso2is --set rbac.create=true
  ```
  
## Quick Start Guide
>In the context of this document, <br>
>* `KUBERNETES_HOME` will refer to a local copy of the [`wso2/kubernetes-is`](https://github.com/wso2/kubernetes-is/)
Git repository. <br>
>* `HELM_HOME` will refer to `<KUBERNETES_HOME>/helm/is`. <br>

##### 1. Clone the Kubernetes Resources for WSO2 Identity Server Git repository.

```
git clone https://github.com/wso2/kubernetes-is.git
```

##### 2. Setup a Network File System (NFS) to be used for persistent storage.

Create and export unique directories within the NFS server instance for each of the following Kubernetes Persistent Volume
resources defined in the `<HELM_HOME>/is-conf/values.yaml` file:

* `sharedDeploymentLocationPath`
* `sharedTenantsLocationPath`

Grant ownership to `wso2carbon` user and `wso2` group, for each of the previously created directories.

  ```
  sudo chown -R wso2carbon:wso2 <directory_name>
  ```

Grant read-write-execute permissions to the `wso2carbon` user, for each of the previously created directories.

  ```
  chmod -R 700 <directory_name>
  ```

##### 3. Provide configurations.

a. The default product configurations are available at `<HELM_HOME>/is-conf/confs` folder. Change the 
configurations as necessary.

b. Open the `<HELM_HOME>/is-conf/values.yaml` and provide the following values.

| Parameter                       | Description                                                                               |
|---------------------------------|-------------------------------------------------------------------------------------------|
| `username`                      | Your WSO2 username                                                                        |
| `password`                      | Your WSO2 password                                                                        |
| `email`                         | Docker email                                                                              |
| `namespace`                     | Kubernetes Namespace in which the resources are deployed                                  |
| `svcaccount`                    | Kubernetes Service Account in the `namespace` to which product instance pods are attached |
| `serverIp`                      | NFS Server IP                                                                             |
| `sharedDeploymentLocationPath`  | NFS shared deployment directory(`<IS_HOME>/repository/deployment`) location for EI        |
| `sharedTenantsLocationPath`     | NFS shared tenants directory(`<IS_HOME>/repository/tenants`) location for EI              |

c. Open the `<HELM_HOME>/is-deployment/values.yaml` and provide the following values. 
    
| Parameter                       | Description                                                                               |
|---------------------------------|-------------------------------------------------------------------------------------------|
| `namespace`                     | Kubernetes Namespace in which the resources are deployed                                  |
| `svcaccount`                    | Kubernetes Service Account in the `namespace` to which product instance pods are attached |

##### 4. Deploy the configurations.

```
helm install --name <RELEASE_NAME> <HELM_HOME>/is-conf
```

##### 5. Deploy product database(s) using MySQL in Kubernetes.

```
helm install --name wso2is-rdbms-service -f <HELM_HOME>/mysql/values.yaml stable/mysql --namespace <NAMESPACE>
```

`NAMESPACE` should be same as in `step 3.b`.

For a serious deployment (e.g. production grade setup), it is recommended to connect product instances to a user owned and managed RDBMS instance.

##### 6. Deploy WSO2 Identity server.

```
helm install --name <RELEASE_NAME> <HELM_HOME>/is-deployment
```

##### 7. Access Management Console.

Default deployment will expose `wso2is` host (to expose Administrative services and Management Console).

To access the console in the environment,

a. Obtain the external IP (`EXTERNAL-IP`) of the Ingress resources by listing down the Kubernetes Ingresses.

```
kubectl get ing
```

```
NAME                       HOSTS          ADDRESS        PORTS     AGE
wso2is-ingress             wso2is         <EXTERNAL-IP>  80, 443   3m
```

b. Add the above host as an entry in /etc/hosts file as follows:

```
<EXTERNAL-IP>	wso2is
```

c. Try navigating to `https://wso2is/carbon` from your favorite browser.
