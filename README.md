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
$ SOLR_HOST=$(kubectl get pods -l app=solr-pod --template "{{ range .items }}{{if eq .status.phase \"Running\" }}{{ .status.hostIP }}{{ end }}{{ end }}") && \
    echo ${SOLR_HOST}
172.17.4.202

$ SOLR_PORT=$(kubectl get services -l app=solr-service --template "{{ range .items }}{{ range .spec.ports }}{{ if eq .name \"solr\" }}{{ .nodePort }}{{ end }}{{ end }}{{ end }}") && \
    echo ${SOLR_PORT}
30647
```

### 8. Create core

```sh
$ curl "http://${SOLR_HOST}:${SOLR_PORT}/solr/admin/cores?action=CREATE&name=collection1&configSet=data_driven_schema_configs&dataDir=data" | \
    xmllint --format -
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   184    0   184    0     0     39      0 --:--:--  0:00:04 --:--:--    39
<?xml version="1.0" encoding="UTF-8"?>
<response>
  <lst name="responseHeader">
    <int name="status">0</int>
    <int name="QTime">4621</int>
  </lst>
  <str name="core">collection1</str>
</response>
```

### 9. Open Solr Admin UI in a browser

Open Solr Admin UI ([http://172.17.4.202:30647/solr/#/](http://172.17.4.202:30647/solr/#/)) in a browser.



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
$ kubectl create -f ${HOME}/git/kubernetes-solr/5.5/solr-service-1.yaml
You have exposed your service on an external port on all nodes in your
cluster.  If you want to expose this service to the external internet, you may
need to set up firewall rules for the service port(s) (tcp:31946,tcp:32766,tcp:31129) to serve traffic.

See http://releases.k8s.io/release-1.1/docs/user-guide/services-firewalls.md for more details.
service "solr-service-1" created

$ kubectl create -f ${HOME}/git/kubernetes-solr/5.5/solr-service-2.yaml
You have exposed your service on an external port on all nodes in your
cluster.  If you want to expose this service to the external internet, you may
need to set up firewall rules for the service port(s) (tcp:30731,tcp:31027,tcp:30260) to serve traffic.

See http://releases.k8s.io/release-1.1/docs/user-guide/services-firewalls.md for more details.
service "solr-service-2" created

$ kubectl create -f ${HOME}/git/kubernetes-solr/5.5/solr-service-3.yaml
You have exposed your service on an external port on all nodes in your
cluster.  If you want to expose this service to the external internet, you may
need to set up firewall rules for the service port(s) (tcp:30933,tcp:31172,tcp:32132) to serve traffic.

See http://releases.k8s.io/release-1.1/docs/user-guide/services-firewalls.md for more details.
service "solr-service-3" created

$ kubectl create -f ${HOME}/git/kubernetes-solr/5.5/solr-service-4.yaml
You have exposed your service on an external port on all nodes in your
cluster.  If you want to expose this service to the external internet, you may
need to set up firewall rules for the service port(s) (tcp:31848,tcp:30953,tcp:30658) to serve traffic.

See http://releases.k8s.io/release-1.1/docs/user-guide/services-firewalls.md for more details.
service "solr-service-4" created
```

### 4. Check Solr servicers

```sh
$ kubectl get services -l group=solr-service
NAME             CLUSTER_IP   EXTERNAL_IP   PORT(S)                       SELECTOR         AGE
solr-service-1   10.3.0.206   nodes         8983/TCP,7983/TCP,18983/TCP   app=solr-pod-1   1m
solr-service-2   10.3.0.200   nodes         8983/TCP,7983/TCP,18983/TCP   app=solr-pod-2   59s
solr-service-3   10.3.0.240   nodes         8983/TCP,7983/TCP,18983/TCP   app=solr-pod-3   40s
solr-service-4   10.3.0.235   nodes         8983/TCP,7983/TCP,18983/TCP   app=solr-pod-4   16s
```

### 5. Start Solr controllers

```sh
$ kubectl create -f ${HOME}/git/kubernetes-solr/5.5/solr-controller-1.yaml
replicationcontroller "solr-controller-1" created

$ kubectl create -f ${HOME}/git/kubernetes-solr/5.5/solr-controller-2.yaml
replicationcontroller "solr-controller-2" created

$ kubectl create -f ${HOME}/git/kubernetes-solr/5.5/solr-controller-3.yaml
replicationcontroller "solr-controller-3" created

$ kubectl create -f ${HOME}/git/kubernetes-solr/5.5/solr-controller-4.yaml
replicationcontroller "solr-controller-4" created
```

### 6. Check Solr controllers

```sh
$ kubectl get replicationcontrollers -l group=solr-controller
CONTROLLER          CONTAINER(S)       IMAGE(S)                         SELECTOR         REPLICAS   AGE
solr-controller-1   solr-container-1   mosuka/docker-solr:release-5.5   app=solr-pod-1   1          1m
solr-controller-2   solr-container-2   mosuka/docker-solr:release-5.5   app=solr-pod-2   1          45s
solr-controller-3   solr-container-3   mosuka/docker-solr:release-5.5   app=solr-pod-3   1          32s
solr-controller-4   solr-container-4   mosuka/docker-solr:release-5.5   app=solr-pod-4   1          20s
```

### 7. Check Solr pods

```sh
$ kubectl get pods -l group=solr-pod -o wide
NAME                      READY     STATUS    RESTARTS   AGE       NODE
solr-controller-1-cu2f8   1/1       Running   0          1m        172.17.4.202
solr-controller-2-yeh1u   1/1       Running   0          1m        172.17.4.202
solr-controller-3-gi4i2   1/1       Running   0          54s       172.17.4.202
solr-controller-4-9b8t1   1/1       Running   0          48s       172.17.4.202
```

### 8. Get host IP and port

```sh
$ SOLR_1_HOST=$(kubectl get pods -l app=solr-pod-1 --template "{{ range .items }}{{if eq .status.phase \"Running\" }}{{ .status.hostIP }}{{ end }}{{ end }}") && \
    echo ${SOLR_1_HOST}
172.17.4.202

$ SOLR_1_PORT=$(kubectl get services -l app=solr-service-1 --template "{{ range .items }}{{ range .spec.ports }}{{ if eq .name \"solr\" }}{{ .nodePort }}{{ end }}{{ end }}{{ end }}") && \
    echo ${SOLR_1_PORT}
31946

$ SOLR_2_HOST=$(kubectl get pods -l app=solr-pod-2 --template "{{ range .items }}{{if eq .status.phase \"Running\" }}{{ .status.hostIP }}{{ end }}{{ end }}") && \
    echo ${SOLR_2_HOST}
172.17.4.201

$ SOLR_2_PORT=$(kubectl get services -l app=solr-service-2 --template "{{ range .items }}{{ range .spec.ports }}{{ if eq .name \"solr\" }}{{ .nodePort }}{{ end }}{{ end }}{{ end }}") && \
    echo ${SOLR_2_PORT}
30731

$ SOLR_3_HOST=$(kubectl get pods -l app=solr-pod-3 --template "{{ range .items }}{{if eq .status.phase \"Running\" }}{{ .status.hostIP }}{{ end }}{{ end }}") && \
    echo ${SOLR_3_HOST}
172.17.4.201

$ SOLR_3_PORT=$(kubectl get services -l app=solr-service-3 --template "{{ range .items }}{{ range .spec.ports }}{{ if eq .name \"solr\" }}{{ .nodePort }}{{ end }}{{ end }}{{ end }}") && \
    echo ${SOLR_3_PORT}
30933

$ SOLR_4_HOST=$(kubectl get pods -l app=solr-pod-4 --template "{{ range .items }}{{if eq .status.phase \"Running\" }}{{ .status.hostIP }}{{ end }}{{ end }}") && \
    echo ${SOLR_4_HOST}
172.17.4.201

$ SOLR_4_PORT=$(kubectl get services -l app=solr-service-4 --template "{{ range .items }}{{ range .spec.ports }}{{ if eq .name \"solr\" }}{{ .nodePort }}{{ end }}{{ end }}{{ end }}") && \
    echo ${SOLR_4_PORT}
31848

### 9. Get /live_nodes

```sh
$ LIVE_NODES=$(echo $(curl "http://${SOLR_1_HOST}:${SOLR_1_PORT}/solr/admin/zookeeper?detail=true&path=%2Flive_nodes" | jq -r '.tree[].children[].data.title') | sed -e s'/ /,/g')
$ echo ${LIVE_NODES}
solr-service-1:8983_solr,solr-service-2:8983_solr,solr-service-3:8983_solr,solr-service-4:8983_solr
```

### 9. Create collection

```sh
$ curl "http://${SOLR_1_HOST}:${SOLR_1_PORT}/solr/admin/collections?action=CREATE&name=collection1&numShards=2&replicationFactor=2&maxShardsPerNode=1&createNodeSet=${LIVE_NODES}&collection.configName=data_driven_schema_configs" | \
    xmllint --format -
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   184    0   184    0     0     39      0 --:--:--  0:00:04 --:--:--    39
<?xml version="1.0" encoding="UTF-8"?>
<response>
  <lst name="responseHeader">
    <int name="status">0</int>
    <int name="QTime">4621</int>
  </lst>
  <str name="core">collection1</str>
</response>
```
