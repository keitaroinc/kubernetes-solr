# kubernetes-solr
Kubernetes resource for Apache Solr.

## Standalone Solr example

### 1. Clone kubernetes-solr

```sh
$ git clone https://github.com/mosuka/kubernetes-solr.git
```

### 2. Start Solr service

```sh
$ $ kubectl create -f $HOME/git/kubernetes-solr/5.5/solr-service.yaml
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
$ kubectl create -f $HOME/git/kubernetes-solr/5.5/solr-controller.yaml
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
$ kubectl get pods -l app=solr-pod --template "{{ range .items }}{{if eq .status.phase \"Running\" }}{{ .status.hostIP }}{{ end }}{{ end }}"
172.17.4.202

$ kubectl get services -l app=solr-service --template "{{ range .items }}{{ range .spec.ports }}{{ if eq .name \"solr\" }}{{ .nodePort }}{{ end }}{{ end }}{{ end }}"
30647
```

### 8. Create core

```sh
$ curl "http://172.17.4.202:30647/solr/admin/cores?action=CREATE&name=collection1&configSet=data_driven_schema_configs&dataDir=data" | \
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

