# Kubeadm CI Container
[![Docker Repository on Quay](https://quay.io/repository/attcomdev/kubeadm-ci/status "Docker Repository on Quay")](https://quay.io/repository/attcomdev/kubeadm-ci)

This continer is intended to be used in CI. It takes the concepts from [kubeadm issue #17](https://github.com/kubernetes/kubeadm/issues/17), and the recommendations from @mikedanese.

## Instructions

To use this container, use these simple instructions:

**Run the container:**
```
docker run -it -e "container=docker" --privileged=true --name kubeadm-ci -d --security-opt seccomp:unconfined --cap-add=SYS_ADMIN -v /sys/fs/cgroup:/sys/fs/cgroup:ro -v /var/run/docker.sock:/var/run/docker.sock  quay.io/attcomdev/kubeadm-ci:latest /sbin/init
```

**Enter the container:**
```
docker exec -it kubeadm-ci /bin/bash
```

**Configure the container:**
```
echo "Updating Ubuntu..."
apt-get update -y
apt-get upgrade -y

systemctl start docker

echo "Install os requirements"
apt-get install -y \
  curl \
  apt-transport-https \
  dialog \
  python \
  daemon

echo "Add Kubernetes repo..."
sh -c 'curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -'
sh -c 'echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list'
apt-get update -y

echo "Installing Kubernetes requirements..."
apt-get install -y \
  kubelet

# This is temporary fix until new version will be released
sed -i 38,40d /var/lib/dpkg/info/kubelet.postinst

apt-get install -y \
  kubernetes-cni \
  kubectl \
  kubeadm
```

Now you can `kubeadm init --skip-preflight-checks` the container as you would normally on any system.
