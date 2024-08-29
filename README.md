# k8s-learn

# rocky linux 8.6 rke2 and rancher install script
```
dnf update
```
# クラスタで一意になるように設定する
```
hostnamectl --set-hostname rke2-localhost
```

```
vi /etc/sysconfig/selinux
-> SELINUX=disabled
```
```
systemctl stop firewalld
systemctl disable firewalld
reboot
```
```
vi /etc/NetworkManager/conf.d/rke2-canal.conf
->
[keyfile]
unmanaged-devices=interface-name:cali*;interface-name:flannel*
```
```
systemctl reload NetworkManager
```

```
mkdir /root/rke2-artifacts && cd /root/rke2-artifacts/
curl -OLs https://github.com/rancher/rke2/releases/download/v1.26.10%2Brke2r2/rke2-images.linux-amd64.tar.zst
curl -OLs https://github.com/rancher/rke2/releases/download/v1.26.10%2Brke2r2/rke2.linux-amd64.tar.gz
curl -OLs https://github.com/rancher/rke2/releases/download/v1.26.10%2Brke2r2/sha256sum-amd64.txt
curl -sfL https://get.rke2.io --output install.sh
INSTALL_RKE2_ARTIFACT_PATH=/root/rke2-artifacts sh install.sh
```

# rke2の実行（初回は30分ぐらい待つと勝手にstatusがactiveになる）
```
systemctl enable rke2-server.service
systemctl start rke2-server.service
```

# kubectlの設定
```
vi ~/.bashrc
->
export KUBECONFIG=/etc/rancher/rke2/rke2.yaml
export PATH=$PATH:/var/lib/rancher/rke2/bin/
```

```
kubectl get node -o wide
```

# helmのインストール
```
mkdir /root/helm
cd /root/helm/
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

# rancherのインストール
```
helm repo add rancher-stable https://releases.rancher.com/server-charts/stable

kubectl create namespace cattle-system

kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.12.13/cert-manager.crds.yaml

helm repo add jetstack https://charts.jetstack.io

helm repo update

helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace
  
helm install rancher rancher-stable/rancher \
  --namespace cattle-system \
  --set hostname=your_host_name_here.local \
  --set replicas=3 \
  --set bootstrapPassword=your_password_here \
  --version=2.4.8
```


