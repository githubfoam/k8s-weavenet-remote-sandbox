# kubernetes sandbox

cross platform(freebsd,lin,win,mac..etc)

~~~~
config.vm.define "k8s-master01" do |k8scluster|
    k8scluster.vm.box = "bento/ubuntu-19.04"
config.vm.define "worker01" do |k8scluster|
    k8scluster.vm.box = "bento/ubuntu-16.04"
config.vm.define "worker02" do |k8scluster|
    k8scluster.vm.box = "bento/ubuntu-18.10"
config.vm.define "remotecontrol01" do |k8scluster|
    k8scluster.vm.box = "bento/centos-7.7"

>vagrant global-status
id       name            provider   state    directory
-----------------------------------------------------------------------------------------------------------
c34c93c  k8s-master01    virtualbox running  C:/multimachine/kubernetes-sandbox-remote
adb4ffe  worker01        virtualbox running  C:/multimachine/kubernetes-sandbox-remote
2e21187  worker02        virtualbox running  C:/multimachine/kubernetes-sandbox-remote
b39b49d  remotecontrol01 virtualbox running  C:/multimachine/kubernetes-sandbox-remote


>vagrant ssh remotecontrol01

vagrant@remotecontrol01:~$ sudo ansible-playbook -i /vagrant/kube-cluster/hosts /vagrant/kube-cluster/1_initial.yml
vagrant@remotecontrol01:~$ sudo ansible-playbook -i /vagrant/kube-cluster/hosts /vagrant/kube-cluster/2_kube-dependencies.yml
vagrant@remotecontrol01:~$ sudo ansible-playbook -i /vagrant/kube-cluster/hosts /vagrant/kube-cluster/3_masters.yml
vagrant@remotecontrol01:~$ sudo ansible-playbook -i /vagrant/kube-cluster/hosts /vagrant/kube-cluster/4_workers.yml
~~~~


~~~~
>vagrant ssh k8s-master01

vagrant@k8s-master01:~$ kubectl get nodes
NAME           STATUS   ROLES    AGE     VERSION
k8s-master01   Ready    master   9m9s    v1.15.2
worker01       Ready    <none>   6m26s   v1.15.2
worker02       Ready    <none>   5m49s   v1.15.2

vagrant@k8s-master01:~$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                   READY   STATUS    RESTARTS   AGE
kube-system   coredns-5c98db65d4-7gsl8               1/1     Running   0          27m
kube-system   coredns-5c98db65d4-cm5ds               1/1     Running   0          27m
kube-system   etcd-k8s-master01                      1/1     Running   0          27m
kube-system   kube-apiserver-k8s-master01            1/1     Running   0          27m
kube-system   kube-controller-manager-k8s-master01   1/1     Running   0          27m
kube-system   kube-proxy-b26z8                       1/1     Running   0          24m
kube-system   kube-proxy-hzq7h                       1/1     Running   0          25m
kube-system   kube-proxy-wpflx                       1/1     Running   0          27m
kube-system   kube-scheduler-k8s-master01            1/1     Running   0          27m
kube-system   weave-net-4dp42                        2/2     Running   0          25m
kube-system   weave-net-7rr76                        2/2     Running   1          24m
kube-system   weave-net-ckth8                        2/2     Running   0          27m

vagrant@k8s-master01:~$ kubectl logs -n kube-system weave-net-4dp42 weave

Install the weave script
vagrant@k8s-master01:~$ sudo curl -L git.io/weave -o /usr/local/bin/weave
vagrant@k8s-master01:~$ sudo chmod a+x /usr/local/bin/weave
vagrant@k8s-master01:~$ sudo weave status

vagrant@k8s-master01:~$ kubectl get pods -n kube-system -l name=weave-net -o wide
NAME              READY   STATUS    RESTARTS   AGE   IP              NODE           NOMINATED NODE   READINESS GATES
weave-net-4dp42   2/2     Running   0          30m   192.168.50.11   worker01       <none>           <none>
weave-net-7rr76   2/2     Running   1          29m   192.168.50.12   worker02       <none>           <none>
weave-net-ckth8   2/2     Running   0          33m   192.168.50.10   k8s-master01   <none>           <none>

shows all Weave Net pods available
vagrant@k8s-master01:~$ kubectl get pods --all-namespaces | grep weave
kube-system   weave-net-89ckd                        2/2     Running   0          50m
kube-system   weave-net-bvkbw                        2/2     Running   1          50m
kube-system   weave-net-m6rd2                        2/2     Running   0          57m

~~~~
upgrade kubernetes
~~~~
vagrant ssh k8s-master01
$ kubectl get nodes


$ apt-cache madison kubelet | more
 kubelet |  1.16.2-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages

 vagrant@k8s-master01:~$ apt-cache madison kubelet | more
    kubelet |  1.17.2-00 | http://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
    kubelet |  1.17.1-00 | http://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
    kubelet |  1.17.0-00 | http://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
    kubelet |  1.16.6-00 | http://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
    kubelet |  1.16.5-00 | http://apt.kubernetes.io kubernetes-xenial/main amd64 Packages

#kubernetes-sandbox-remote-calico\kube-cluster\2_kube-dependencies.yml
kubernetes_version : "=1.17.0-00"
validated_dockerv: "=5:19.03.4~3-0~ubuntu-xenial"

https://kubernetes.io/docs/setup/release/notes/

~~~~
upgrade weavenet
~~~~
Installing a pod network add-on
kubectl apply -f https://docs.projectcalico.org/v3.8/manifests/calico.yaml

$ kubectl get pods -n kube-system
$ kubectl -n kube-system logs coredns-5644d7b6d9-cw9bg

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm

Network plugins in Kubernetes come in a few flavors:
CNI plugins: adhere to the appc/CNI specification, designed for interoperability.
Kubenet plugin: implements basic cbr0 using the bridge and host-local CNI plugins
The kubelet has a single default network plugin, and a default network common to the entire cluster. It probes for plugins when it starts up, remembers what it finds, and executes the selected plugin at appropriate times in the pod lifecycle (this is only true for Docker, as rkt manages its own CNI plugins).
https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/#network-plugin-requirements

~~~~

upgrade docker
~~~~  

vagrant@k8s-master01:~$ apt-cache madison docker-ce
 docker-ce | 5:19.03.5~3-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
 docker-ce | 5:19.03.4~3-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
 docker-ce | 5:19.03.3~3-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
 docker-ce | 5:19.03.2~3-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
 docker-ce | 5:19.03.1~3-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages


Update the latest validated version of Docker to 19.03 (#84476, @neolit123)
https://kubernetes.io/docs/setup/release/notes/

On each of your machines, install Docker. Version 19.03.4 is recommended, but 1.13.1, 17.03, 17.06, 17.09, 18.06 and 18.09 are known to work as well. Keep track of the latest verified Docker version in the Kubernetes release notes.
https://kubernetes.io/docs/setup/production-environment/container-runtimes/

[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/

~~~~

Controlling your cluster from machines other than the control-plane node
~~~~
vagrant@k8s-master01:~$ hostnamectl | grep "Operating System"
  Operating System: Ubuntu 19.04
vagrant@worker01:~$ hostnamectl | grep "Operating System"
    Operating System: Ubuntu 16.04.5 LTS
vagrant@worker02:~$ hostnamectl | grep "Operating System"
      Operating System: Ubuntu 18.10


[vagrant@remotecontrol01 ~]$ cat /etc/redhat-release
CentOS Linux release 7.6.1810 (Core)


@k8s-master01:/etc/kubernetes/admin.conf .
copy the administrator kubeconfig file from your control-plane node to your workstation
scp vagrant@k8s-master01:/~admin.conf .

Install kubectl
[vagrant@remotecontrol01 ~]$ cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
        https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
[vagrant@remotecontrol01 ~]$ sudo yum install -y kubectl
[vagrant@remotecontrol01 ~]$ kubectl --kubeconfig ./admin.conf get nodes
NAME           STATUS   ROLES    AGE   VERSION
k8s-master01   Ready    master   38m   v1.15.2
worker01       Ready    <none>   31m   v1.15.2
worker02       Ready    <none>   30m   v1.15.2

~~~~
