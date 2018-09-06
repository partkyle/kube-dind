Example of docker in docker in kubernetes
=========================================

This is an example of docker in docker in a kubernetes cluster.

Important points to note:

- This requires running the docker in docker (dind) container in `privileged` mode.
- This requires a large amount of disk storage for the docker images that are downloaded.
- The secondary container in the pod is used as an example of how to communicate between the dind container
  and the container where you want to initiate docker commads from.
- This will be isolated from the docker running that manages kubernetes

Examples:

Create the kube deployment:

```
~/s/kube-dind > kubectl apply -f deployment.yaml
deployment.apps/dind created
```

The pod should report running
```
~/s/kube-dind > kubectl get all
NAME                        READY     STATUS    RESTARTS   AGE
pod/dind-7679579dd5-8gkmj   2/2       Running   0          5m

NAME                     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE
service/kubernetes       ClusterIP   10.96.0.1        <none>        443/TCP     7d

NAME                   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/dind   1         1         1            1           5m

NAME                              DESIRED   CURRENT   READY     AGE
replicaset.apps/dind-7679579dd5   1         1         1         5m
```

Now you can exec into the idle container running alipine linux and execute docker commands.

```
/ # apk add --update docker
fetch http://dl-cdn.alpinelinux.org/alpine/v3.8/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.8/community/x86_64/APKINDEX.tar.gz
(1/9) Installing ca-certificates (20171114-r3)
(2/9) Installing libmnl (1.0.4-r0)
(3/9) Installing jansson (2.11-r0)
(4/9) Installing libnftnl-libs (1.1.1-r0)
(5/9) Installing iptables (1.6.2-r0)
(6/9) Installing device-mapper-libs (2.02.178-r0)
(7/9) Installing libltdl (2.4.6-r5)
(8/9) Installing libseccomp (2.3.3-r1)
(9/9) Installing docker (18.06.1-r0)
Executing docker-18.06.1-r0.pre-install
Executing busybox-1.28.4-r0.trigger
Executing ca-certificates-20171114-r3.trigger
OK: 182 MiB in 22 packages
/ # docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

Note that this is isolated from the docker that is managing kubernetes.

You should now be able to run containers as you wish:

```
(mac) > kubectl exec -it dind-7679579dd5-8gkmj -c sh sh
/ # docker run -itd alpine sh
Unable to find image 'alpine:latest' locally
latest: Pulling from library/alpine
8e3ba11ec2a2: Pull complete
Digest: sha256:7043076348bf5040220df6ad703798fd8593a0918d06d3ce30c6c93be117e430
Status: Downloaded newer image for alpine:latest
dcc2c6356e65717cf25a25c25fd9287484e5abb96be51fb264b0a6d56c15df4d
/ # docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
dcc2c6356e65        alpine              "sh"                3 seconds ago       Up 2 seconds                            kind_feynman
/ # docker run alpine env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=4802154d706b
HOME=/root
/ # docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
4802154d706b        alpine              "env"               9 seconds ago       Exited (0) 8 seconds ago                        flamboyant_minsky
dcc2c6356e65        alpine              "sh"                55 seconds ago      Exited (0) 19 seconds ago                       kind_feynman
```

note that this is different that the host docker by checking the status of docker ps -a on the host machine.

You can also remove containers with reckless abandon, because they are isolated to this instance of docker running inside of the container

```
/ # docker rm -fv $(docker ps -qa)
4802154d706b
dcc2c6356e65
/ # docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

