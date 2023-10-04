![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/multi_container&empty)
# Multi-container Pods (10%)

### Create a Pod with two containers, both with image busybox and command "echo hello; sleep 3600". Connect to the second container, named tools and run 'ls'

<details><summary>show</summary>
<p>

Easiest way to do it is create a pod with a single container and save its definition in a YAML file:

```bash
kubectl run busybox --image=busybox --restart=Never -o yaml --dry-run=client -- /bin/sh -c 'echo hello;sleep 3600' > pod1.yaml
vi pod.yaml
```

Copy/paste the container related values, so your final YAML should contain the following two containers (make sure those containers have a different name):

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  containers:
  - args:
    - /bin/sh
    - -c
    - echo hello;sleep 3600
    image: busybox
    name: busybox
    resources: {}
  - args:
    - /bin/sh
    - -c
    - sleep 3600
    image: busybox
    name: tools
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod1.yaml
# Connect to the tools container within the pod
kubectl exec -it busybox -c tools -- /bin/sh
ls
exit

# or you can do the above with just an one-liner
kubectl exec -it busybox -c tools -- ls

# you can do some cleanup
kubectl delete po busybox
```

</p>
</details>

### Create a pod with an nginx container exposed on port 80. Add a busybox init container which downloads a page using "wget -O /work-dir/index.html https://www.ifconfig.io/all.json". Make a volume of type emptyDir and mount it in both containers. For the nginx container, mount it on "/usr/share/nginx/html" and for the initcontainer, mount it on "/work-dir". When done, get the IP of the created pod and create a busybox pod and run "wget -O- IP using "

<details><summary>show</summary>
<p>

Easiest way to do it is create a pod with a single container and save its definition in a YAML file:

```bash
kubectl run box --image=nginx --restart=Never --port=80 --dry-run=client -o yaml > pod-init.yaml
```

Copy/paste the container related values, so your final YAML should contain the volume and the initContainer:

Volume:

```YAML
containers:
- image: nginx
...
  volumeMounts:
  - name: vol1
    mountPath: /usr/share/nginx/html
volumes:
- name: vol1
  emptyDir: {}
```

initContainer:

```YAML
...
  initContainers:
  - name: box
    image: busybox
    args:
    - /bin/sh
    - -c
    - "wget -O /work-dir/index.html https://www.ifconfig.io/all.json"
    volumeMounts:
    - name: vol1
      mountPath: /work-dir
```

In total you get:

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: box
  name: box
spec:
  initContainers:
  - name: box
    image: busybox
    args:
    - /bin/sh
    - -c
    - "wget -O /work-dir/index.html https://www.ifconfig.io/all.json"
    volumeMounts:
    - name: vol1
      mountPath: /work-dir
  containers:
  - image: nginx
    name: nginx
    ports:
    - containerPort: 80
    resources: {}
    volumeMounts:
    - name: vol1
      mountPath: /usr/share/nginx/html
  dnsPolicy: ClusterFirst
  restartPolicy: Never
  volumes:
  - name: vol1
    emptyDir: {}
status: {} 
```

```bash
# Apply pod
kubectl apply -f pod-init.yaml

# Get IP
kubectl get po -o wide

# Execute wget
kubectl run box-test --image=busybox --restart=Never -it --rm -- /bin/sh -c "wget -O- 192.168.1.4"
#or
kubectl run box-test --image=busybox --restart=Never -it --rm -- /bin/sh -c "wget -O- $(kubectl get pod box -o jsonpath='{.status.podIP}')"
```bash
Connecting to 192.168.1.4 (192.168.1.4:80)
writing to stdout
{"country_code":"US","encoding":"gzip","forwarded":"51.81.93.100","host":"51.81.93.100","ifconfig_cmd_hostname":"ifconfig.io","ifconfig_hostname":"ifconfig.io","ip":"51.81.93.100","lang":"","method":"GET","mime":"","port":47588,-                    100% |********************************|   253  0:00:00 ETA
written to stdout
pod "box-teest" deleted
```


# you can do some cleanup
kubectl delete po box
```

</p>
</details>