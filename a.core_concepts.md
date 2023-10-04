![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/core_concepts&empty)
# Core Concepts (13%)

kubernetes.io > Documentation > Reference > kubectl CLI > [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

kubernetes.io > Documentation > Tasks > Monitoring, Logging, and Debugging > [Get a Shell to a Running Container](https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/)

kubernetes.io > Documentation > Tasks > Access Applications in a Cluster > [Configure Access to Multiple Clusters](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)

kubernetes.io > Documentation > Tasks > Access Applications in a Cluster > [Accessing Clusters](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/) using API

kubernetes.io > Documentation > Tasks > Access Applications in a Cluster > [Use Port Forwarding to Access Applications in a Cluster](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/)

### Prepair
```
alias k=kubectl
```


### Create a pod with name 'frontend' with image latest nginx in namespace called 'prod'.

<details><summary>show</summary>
<p>

```bash
kubectl create namespace prod
kubectl run frontend --image=nginx:latest --restart=Never -n prod
kubectl -n prod get pod/frontend
```

</p>
</details>

### Create the pod 'web-app' in namespace 'prod' that was just described using YAML

<details><summary>show</summary>
<p>

Easily generate YAML with:

```bash
kubectl get ns
kubectl create ns prod
kubectl run web-app --image=nginx --restart=Never --dry-run=client -n prod -o yaml > web-app.yaml
```

```bash
cat web-app.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: web-app
  name: web-app
  namespace: prod
spec:
  containers:
  - image: nginx
    name: web-app
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
```

Or, you can run:

```bash
kubectl run web-app --image=nginx --restart=Never --dry-run=client -n prod -o yaml | kubectl create -n mynamespace -f -
```

</p>
</details>

### Create a pod named 'tools' with image busybox (using kubectl command) that runs the command "env" (print envs in this pod). Run it and save the output on /mnt/env-busybox.txt

<details><summary>show</summary>
<p>

```bash
kubectl run tools --image=busybox --command --restart=Never -it --rm -- env # -it will help in seeing the output, --rm will immediately delete the pod after it exits
kubectl run tools --image=busybox --command --restart=Never -it --rm -- env > /mnt/env-busybox.txt
# or, just run it without -it
kubectl run tools --image=busybox --command --restart=Never -- env
# and then, check its logs
kubectl logs tools
```

</p>
</details>

### Create a pod named 'tools' with image busybox (using YAML) that runs the command "env" on namespace 'staging'. Run it and see the output

<details><summary>show</summary>
<p>

```bash
# create a  YAML template namespace with this command
kubectl create ns staging --dry-run=client -o yaml > ns-staging.yaml
# see it
cat ns-staging.yaml
```

```YAML
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: null
  name: staging
spec: {}
```

```bash
# apply
kubectl apply -f ns-staging.yaml
# create a  YAML template pod with this command
kubectl -n staging run tools --image=busybox --restart=Never --dry-run=client -o yaml --command -- env > tools-pod.yaml
# see it
cat tools-pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: tools
  name: tools
  namespace: staging
spec:
  containers:
  - command:
    - env
    image: busybox
    name: tools
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
# apply it and then see the logs
kubectl -n staging apply -f tools-pod.yaml
kubectl -n staging logs tools
```

</p>
</details>

### View the YAML for a new namespace called 'develop' without creating it

<details><summary>show</summary>
<p>

```bash
kubectl create namespace develop -o yaml --dry-run=client
```

</p>
</details>

### Create the YAML for a new ResourceQuota called 'develop-rq', on namespace develop, with hard limits of 2 CPU, 2G memory and 4 pods without creating it

<details><summary>show</summary>
<p>

```bash
kubectl -n develop create quota develop-rq --hard=cpu=2,memory=2G,pod=4 --dry-run=client -o yaml
```

</p>
</details>

### Get deploy,pods and services on all namespaces

<details><summary>show</summary>
<p>

```bash
kubectl get po,deploy,svc --all-namespaces
# or
kubectl get po,deploy,svc -A
```

</p>
</details>

### Create a pod, in staging namespace, with image nginx (version 1.19) called 'web-app' and expose traffic on port 8080

<details><summary>show</summary>
<p>

```bash
kubectl create ns staging
kubectl -n staging run web-app --image=nginx:1.19 --restart=Never --port=8080
kubectl -n staging exec web-app -it -- curl http://web-app
```

</p>
</details>

### Deploy pod' nginx:1.20 and change pod's image to nginx:1.19. Observe that the container will be restarted as soon as the image gets pulled

<details><summary>show</summary>
<p>

*Note*: The `RESTARTS` column should contain 0 initially (ideally - it could be any number)

```bash
kubectl run nginx --image=nginx:1.20 --restart=Never
# kubectl set image POD/POD_NAME CONTAINER_NAME=IMAGE_NAME:TAG
kubectl set image pod/nginx nginx=nginx:1.20
kubectl describe po nginx # you will see an event 'Container will be killed and recreated'
kubectl get po nginx -w # watch it
```

*Note*: some time after changing the image, you should see that the value in the `RESTARTS` column has been increased by 1, because the container has been restarted, as stated in the events shown at the bottom of the `kubectl describe pod` command:

```
Events:
  Type    Reason     Age                  From               Message
  ----    ------     ----                 ----               -------
[...]
  Normal  Scheduled  38s   default-scheduler  Successfully assigned default/nginx to node01
  Normal  Pulling    38s   kubelet            Pulling image "nginx:1.20"
  Normal  Pulled     34s   kubelet            Successfully pulled image "nginx:1.20" in 3.703299337s (3.70330722s including waiting)
  Normal  Created    34s   kubelet            Created container nginx
  Normal  Started    34s   kubelet            Started container nginx
```

*Note*: you can check pod's image by running

```bash
kubectl get po nginx -o jsonpath='{.spec.containers[].image}{"\n"}'
```

</p>
</details>

### Get nginx pod's ip created in previous step, use a temp busybox image to wget its '/'

<details><summary>show</summary>
<p>

```bash
kubectl get po -o wide # get the IP, will be something like '10.1.1.131'
# create a temp busybox pod
kubectl run busybox --image=busybox --rm -it --restart=Never -- wget -O- 10.1.1.131:80
```

Alternatively you can also try a more advanced option:

```bash
# Get IP of the nginx pod
NGINX_IP=$(kubectl get pod nginx -o jsonpath='{.status.podIP}')
# create a temp busybox pod
kubectl run busybox --image=busybox --env="NGINX_IP=$NGINX_IP" --rm -it --restart=Never -- sh -c 'wget -O- $NGINX_IP:80'
``` 

Or just in one line:

```bash
kubectl run busybox --image=busybox --rm -it --restart=Never -- wget -O- $(kubectl get pod nginx -o jsonpath='{.status.podIP}:{.spec.containers[0].ports[0].containerPort}')
```

</p>
</details>

### Get pod's YAML and save on /mnt/nginx.yaml

<details><summary>show</summary>
<p>

```bash
kubectl get po nginx -o yaml > /mnt/nginx.yaml
# or
kubectl get po nginx -oyaml > /mnt/nginx.yaml
# or
kubectl get po nginx --output yaml > /mnt/nginx.yaml
# or
kubectl get po nginx --output=yaml > /mnt/nginx.yaml
```

</p>
</details>

### Get information about the pod, including details about potential issues (e.g. pod hasn't started)

<details><summary>show</summary>
<p>

```bash
kubectl describe po nginx
```

</p>
</details>

### Get pod logs

<details><summary>show</summary>
<p>

```bash
kubectl logs nginx
```

</p>
</details>

### If pod crashed and restarted, get logs about the previous instance

<details><summary>show</summary>
<p>

```bash
kubectl logs nginx -p
# or
kubectl logs nginx --previous
```

</p>
</details>

### Execute a simple shell on the nginx pod

<details><summary>show</summary>
<p>

```bash
kubectl exec -it nginx -- /bin/sh
kubectl delete po nginx
```

</p>
</details>

### Create a busybox pod that echoes 'hello world' and then exits

<details><summary>show</summary>
<p>

```bash
kubectl run busybox --image=busybox -it --restart=Never -- echo 'hello world'
# or
kubectl run busybox --image=busybox -it --restart=Never -- /bin/sh -c 'echo hello world'
kubectl delete po busybox
```

</p>
</details>

### Do the same, but have the pod deleted automatically when it's completed

<details><summary>show</summary>
<p>

```bash
kubectl run busybox --image=busybox -it --rm --restart=Never -- /bin/sh -c 'echo hello world'
kubectl get po # nowhere to be found :)
```

</p>
</details>

### Create an nginx pod and set as envs value as 'var1=val1' and 'var2=val2'. Check the envs values existing within the pod

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never --env=var1=val1 --env=var2=val2
# then
kubectl exec -it nginx -- env
kubectl exec -it nginx -- printenv
# or
kubectl exec -it nginx -- sh -c 'echo $var1'
kubectl exec -it nginx -- sh -c 'echo $var2'
# or
kubectl describe po nginx | grep -A2 Environment:
kubectl delete po nginx
# or
kubectl run nginx --restart=Never --image=nginx --env=var1=val1 --env=var2=val2 -it --rm -- env
# or
kubectl run nginx --image nginx --restart=Never --env=var1=val1 --env=var2=val2 -it --rm -- sh -c 'echo $var1 $var2'
```

</p>
</details>
