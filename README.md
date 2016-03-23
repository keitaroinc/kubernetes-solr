# kubernetes-solr
Kubernetes resource for Apache Solr.

## Standalone Solr example

### 1. Clone kubernetes-solr

```sh
cd ${HOME}
$ git clone https://github.com/mosuka/kubernetes-solr.git
Cloning into 'kubernetes-solr'...
remote: Counting objects: 16, done.
remote: Compressing objects: 100% (14/14), done.
remote: Total 16 (delta 4), reused 10 (delta 2), pack-reused 0
Unpacking objects: 100% (16/16), done.
Checking connectivity... done.
```

### 2. Start Solr service

```sh
$ kubectl create -f ${HOME}/git/kubernetes-solr/5.5/solr-service.yaml
You have exposed your service on an external port on all nodes in your
cluster.  If you want to expose this service to the external internet, you may
need to set up firewall rules for the service port(s) (tcp:32661,tcp:30312,tcp:32347) to serve traffic.

See http://releases.k8s.io/release-1.1/docs/user-guide/services-firewalls.md for more details.
service "solr-service" created
```

### 3. Check Solr service

```sh
$ kubectl get services -l app=solr-service
NAME           CLUSTER_IP   EXTERNAL_IP   PORT(S)                       SELECTOR            AGE
solr-service   10.3.0.130   nodes         8983/TCP,7983/TCP,18983/TCP   app=zookeeper-pod   1m
```

### 4 Start Solr controller

```sh
$ kubectl create -f ${HOME}/git/kubernetes-solr/5.5/solr-controller.yaml
replicationcontroller "solr-controller" created
```

### 5. Check Solr controller

```sh
$ kubectl get replicationcontrollers -l app=solr-controller
CONTROLLER        CONTAINER(S)     IMAGE(S)                         SELECTOR       REPLICAS   AGE
solr-controller   solr-container   mosuka/docker-solr:release-5.5   app=solr-pod   1          1m
```

### 6. Check Solr pod

```sh
$ kubectl get pods -l app=solr-pod
NAME                    READY     STATUS    RESTARTS   AGE
solr-controller-r15v3   0/1       Pending   0          2m
```

### 7. Get host IP and port

```sh
$ SOLR_HOST_LIST=($(kubectl get pods -l app=solr-pod --template "{{ range .items}}{{ if eq .kind \"Pod\" }}{{if eq .status.phase \"Running\" }}{{ .status.hostIP }} {{ end }}{{ end }}{{ end }}"))
$ echo ${SOLR_HOST_LIST[@]}
172.17.4.201
$ SOLR_HOST=${SOLR_HOST_LIST[0]}
$ echo ${SOLR_HOST}

$ SOLR_PORT=$(kubectl get services -l app=solr-service --template "{{ range .items }}{{ range .spec.ports }}{{ if eq .name \"solr\" }}{{ .nodePort }}{{ end }}{{ end }}{{ end }}")
$ echo ${SOLR_PORT}
32661
```

### 8. Open Solr Admin UI in a browser

```sh
$ SOLR_ADMIN_UI=http://${SOLR_HOST}:${SOLR_PORT}/solr/#/
$ echo ${SOLR_ADMIN_UI}
```

Open Solr Admin UI in a browser.



## SolrCloud example

### 1. Start Zookeeper ensemble

See [ZooKeeper ensemble example](https://github.com/mosuka/kubernetes-zookeeper).

### 2. Clone kubernetes-solr

```sh
cd ${HOME}
$ git clone https://github.com/mosuka/kubernetes-solr.git
Cloning into 'kubernetes-solr'...
remote: Counting objects: 16, done.
remote: Compressing objects: 100% (14/14), done.
remote: Total 16 (delta 4), reused 10 (delta 2), pack-reused 0
Unpacking objects: 100% (16/16), done.
Checking connectivity... done.
```

### 3. Start Solr services

```sh
$ kubectl create -f ${HOME}/git/kubernetes-solr/5.5/solr-service.yaml
You have exposed your service on an external port on all nodes in your
cluster.  If you want to expose this service to the external internet, you may
need to set up firewall rules for the service port(s) (tcp:32737,tcp:30762,tcp:30279) to serve traffic.

See http://releases.k8s.io/release-1.1/docs/user-guide/services-firewalls.md for more details.
service "solr-service" created
```

### 4. Check Solr servicers

```sh
$ kubectl get services -l app=solr-service
NAME           CLUSTER_IP   EXTERNAL_IP   PORT(S)                       SELECTOR       AGE
solr-service   10.3.0.142   nodes         8983/TCP,7983/TCP,18983/TCP   app=solr-pod   7s
```

### 5. Start Solr controllers

```sh
$ kubectl create -f ${HOME}/git/kubernetes-solr/5.5/solrcloud-controller.yaml
replicationcontroller "solr-controller" created
```

### 6. Check Solr controllers

```sh
$ kubectl get replicationcontrollers -l app=solr-controller
CONTROLLER        CONTAINER(S)     IMAGE(S)                         SELECTOR       REPLICAS   AGE
solr-controller   solr-container   mosuka/docker-solr:release-5.5   app=solr-pod   4          20s
```

### 7. Check Solr pods

```sh
$ kubectl get pods -l app=solr-pod -o wide
NAME                    READY     STATUS    RESTARTS   AGE       NODE
solr-controller-17uqc   1/1       Running   0          44s       172.17.4.201
solr-controller-1gm74   1/1       Running   0          44s       172.17.4.201
solr-controller-uc0sg   1/1       Running   0          44s       172.17.4.201
solr-controller-v8iqs   1/1       Running   0          44s       172.17.4.201
```

### 8. Get host IP and port

```sh
$ SOLR_HOST_LIST=($(kubectl get pods -l app=solr-pod --template "{{ range .items}}{{ if eq .kind \"Pod\" }}{{if eq .status.phase \"Running\" }}{{ .status.hostIP }} {{ end }}{{ end }}{{ end }}"))
$ echo ${SOLR_HOST_LIST[@]}
172.17.4.201 172.17.4.201 172.17.4.201 172.17.4.201
$ SOLR_HOST=${SOLR_HOST_LIST[0]}
$ echo ${SOLR_HOST}

$ SOLR_PORT=$(kubectl get services -l app=solr-service --template "{{ range .items }}{{ range .spec.ports }}{{ if eq .name \"solr\" }}{{ .nodePort }}{{ end }}{{ end }}{{ end }}")
$ echo ${SOLR_PORT}
32661
```

### 9. Open Solr Admin UI in a browser

```sh
$ SOLR_ADMIN_UI=http://${SOLR_HOST}:${SOLR_PORT}/solr/#/
$ echo ${SOLR_ADMIN_UI}
http://172.17.4.201:32661/solr/#/
```

Open Solr Admin UI in a browser.



