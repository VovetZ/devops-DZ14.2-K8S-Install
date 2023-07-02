# devops-DZ14.2-K8S-Install

# Домашнее задание к занятию «Установка Kubernetes»

### Цель задания

Установить кластер K8s.

### Чеклист готовности к домашнему заданию

1. Развёрнутые ВМ с ОС Ubuntu 20.04-lts.

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Инструкция по установке kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/).
2. [Документация kubespray](https://kubespray.io/).

-----

### Задание 1. Установить кластер k8s с 1 master node

1. Подготовка работы кластера из 5 нод: 1 мастер и 4 рабочие ноды.
2. В качестве CRI — containerd.
3. Запуск etcd производить на мастере.
4. Способ установки выбрать самостоятельно.

## Дополнительные задания (со звёздочкой)

**Настоятельно рекомендуем выполнять все задания под звёздочкой.** Их выполнение поможет глубже разобраться в материале.
Задания под звёздочкой необязательные к выполнению и не повлияют на получение зачёта по этому домашнему заданию.

------

### Задание 2*. Установить HA кластер

1. Установить кластер в режиме HA.
2. Использовать нечётное количество Master-node.
3. Для cluster ip использовать keepalived или другой способ.

### Правила приёма работы

1. Домашняя работа оформляется в своем Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl get nodes`, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

------

# Ответ

1. Зарегистрировался в [Яндекс Облаке](console.cloud.yandex.ru)
2. Создал платёжный аккаунт с промо-кодом  
3. Установил Yandex Cloud (CLI) `yc`  

```bash
[sudo] password for vk: 
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  curl
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 194 kB of archives.
After this operation, 454 kB of additional disk space will be used.
Get:1 http://ru.archive.ubuntu.com/ubuntu jammy-updates/main amd64 curl amd64 7.81.0-1ubuntu1.10 [194 kB]
Fetched 194 kB in 2s (93,1 kB/s)
Selecting previously unselected package curl.
(Reading database ... 257191 files and directories currently installed.)
Preparing to unpack .../curl_7.81.0-1ubuntu1.10_amd64.deb ...
Unpacking curl (7.81.0-1ubuntu1.10) ...
Setting up curl (7.81.0-1ubuntu1.10) ...
Processing triggers for man-db (2.10.2-1) ...
vkadm@vk-mate:~$ curl -sSL https://storage.yandexcloud.net/yandexcloud-yc/install.sh | bash
Downloading yc 0.107.0
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 99.0M  100 99.0M    0     0  9021k      0  0:00:11  0:00:11 --:--:-- 8378k
Yandex Cloud CLI 0.107.0 linux/amd64

yc PATH has been added to your '/home/vk/.bashrc' profile
yc bash completion has been added to your '/home/vk/.bashrc' profile.
Now we have zsh completion. Type "echo 'source /home/vk/yandex-cloud/completion.zsh.inc' >>  ~/.zshrc" to install itTo complete installation, start a new shell (exec -l $SHELL) or type 'source "/home/vk/.bashrc"' in the current one
```  

5. Настроил `yc` (OAuth токен взял [по адресу](https://oauth.yandex.ru/authorize?response_type=token&client_id=1a6990aa636648e9b2ef855fa7bec2fb)), проверил настройки

```bash
vkadm@vk-mate:~$ yc init
Welcome! This command will take you through the configuration process.
Please go to https://oauth.yandex.ru/authorize?response_type=token&client_id=1a6990aa636648e9b2ef855fa7bec2fb in order to obtain OAuth token.

Please enter OAuth token: y0_Ag...
......
vkadm@vk-mate:~$ yc config list
token: y0_Ag....
cloud-id: b1ggid...
folder-id: b1g12il7....
compute-default-zone: ru-central1-a
```

7. Установил токен и параметры в переменные окружения  

```bash
export YC_TOKEN=$(yc iam create-token)
export YC_CLOUD_ID=$(yc config get cloud-id)
export YC_FOLDER_ID=$(yc config get folder-id)
export YC_ZONE=$(yc config get compute-default-zone)
```

8. Сгенерировал SSH ключи на локальной машине  

```bash
vkadm@vk-mate:~$ ssh-keygen -f ~/.ssh/yandex
Generating public/private rsa key pair.
Created directory '/home/vk/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/vk/.ssh/yandex
Your public key has been saved in /home/vk/.ssh/yandex.pub
...
```

9. Создал ресурсы в Яндекс Облаке

- 1 Master node (2 Gb RAM, 2 CPU, 20 Gb SSD)
- 4 Worker node (2 Gb RAM, 2 CPU, 20 Gb SSD)
- OS `Ubuntu 20.04`
- Cгенерированный ранее публичный ключ (`/root/.ssh/yandex.pub`)
- User `vkadm`

```bash
vk@vk-mate:~$ yc compute instance list
+----------------------+-----------+---------------+---------+----------------+---------------+
|          ID          |   NAME    |    ZONE ID    | STATUS  |  EXTERNAL IP   |  INTERNAL IP  |
+----------------------+-----------+---------------+---------+----------------+---------------+
| fhmg0vpr15g7qstj1p67 | worker-02 | ru-central1-a | RUNNING | 51.250.79.71   | 192.168.10.28 |
| fhmicki8e907i8j0tgm5 | worker-04 | ru-central1-a | RUNNING | 158.160.51.105 | 192.168.10.14 |
| fhmm60vv090g4sbded0m | worker-01 | ru-central1-a | RUNNING | 158.160.59.206 | 192.168.10.7  |
| fhmn6oo9jrfcbcc3gtus | master-01 | ru-central1-a | RUNNING | 84.252.130.211 | 192.168.10.16 |
| fhmtql07gfa8tndef7c9 | worker-03 | ru-central1-a | RUNNING | 158.160.50.148 | 192.168.10.18 |
+----------------------+-----------+---------------+---------+----------------+---------------+
```

10. Установил `kubeadm`, `kubelet`, `kubectl`, `containerd` как CRI на Master node

```bash
ssh -i ~/.ssh/yandex vkadm@84.252.130.211
root@master-01:~# apt-get update -y && apt-get install -y apt-transport-https ca-certificates curl
root@master-01:~# mkdir -p /etc/apt/keyrings
root@master-01:~# curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | gpg --dearmor -o /etc/apt/keyrings/kubernetes-archive-keyring.gpg
root@master-01:~# echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | tee /etc/apt/sources.list.d/kubernetes.list
root@master-01:~# apt-get update -y && apt-get install -y kubelet kubeadm kubectl containerd
root@master-01:~# apt-mark hold kubelet kubeadm kubectl containerd
```

11. Включил `IP forward`

```bash
root@master-01:~# modprobe br_netfilter
root@master-01:~# modprobe overlay
root@master-01:~# echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
root@master-01:~# echo "net.bridge.bridge-nf-call-iptables=1" >> /etc/sysctl.conf
root@master-01:~# echo "net.bridge.bridge-nf-call-arptables=1" >> /etc/sysctl.conf
root@master-01:~# echo "net.bridge.bridge-nf-call-ip6tables=1" >> /etc/sysctl.conf
root@master-01:~# sysctl -p /etc/sysctl.conf
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-arptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
root@master-01:~# cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
> overlay
> br_netfilter
> EOF
overlay
br_netfilter
```

12. Установил Master node

```bash
root@master-01:~# kubeadm init --apiserver-advertise-address=192.168.10.16 --pod-network-cidr 10.244.0.0/16  --apiserver-cert-extra-sans=84.252.130.211,master-01.ru-central1.internal
[init] Using Kubernetes version: v1.27.3
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
....
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.10.16:6443 --token yc7ruq.tbsyzzrij9lnu9a9 \
	--discovery-token-ca-cert-hash sha256:1fcca765ac3012a2ef01bcfc8aeb89427bd01f1afbb9ed93445ee367f7bd91ce 
```

**Запомнили команду ввода Worker node в кластер**

14. Создал `kubeconfig` на Master node

```bash
root@master-01:~# mkdir -p $HOME/.kube
root@master-01:~# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
root@master-01:~# sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

13. Установил сетевой плагин на Master node

```bash
root@master-01:~# kubectl apply -f https://github.com/coreos/flannel/raw/master/Documentation/kube-flannel.yml
namespace/kube-flannel created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.apps/kube-flannel-ds created
```

Далее настройка Worker node. Начнем с `worker-01`

15. Установил `kubeadm`, `kubelet`, `kubectl`, `containerd` как CRI на Worker node

```bash
ssh -i ~/.ssh/yandex vkadm@192.168.10.7

apt-get update && apt-get install -y apt-transport-https ca-certificates curl
mkdir -p /etc/apt/keyrings
curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | gpg --dearmor -o /etc/apt/keyrings/kubernetes-archive-keyring.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | tee /etc/apt/sources.list.d/kubernetes.list
apt-get update && apt-get install -y kubelet kubeadm kubectl containerd
apt-mark hold kubelet kubeadm kubectl containerd
```

16. Включил IP forward

```bash
modprobe br_netfilter
modprobe overlay
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-iptables=1" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-arptables=1" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-ip6tables=1" >> /etc/sysctl.conf
sysctl -p /etc/sysctl.conf
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
```

17. Ввел ноду в кластер (команда из вывода при создании Master node)

```bash
root@worker-01:~# kubeadm join 192.168.10.16:6443 --token yc7ruq.tbsyzzrij9lnu9a9 --discovery-token-ca-cert-hash sha256:1fcca765ac3012a2ef01bcfc8aeb89427bd01f1afbb9ed93445ee367f7bd91ce 
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```

18. Повторил пп. 15-17 для `worker-02`, `worker-03`,`worker-04`

19. Проверил состояние ресурсов кластера (на Master node)

```bash
root@master-01:~# kubectl get nodes
NAME        STATUS   ROLES           AGE     VERSION
master-01   Ready    control-plane   15m     v1.27.3
worker-01   Ready    <none>          7m22s   v1.27.3
worker-02   Ready    <none>          4m28s   v1.27.3
worker-04   Ready    <none>          47s     v1.27.3
worker-03   Ready    <none>          2m23s   v1.27.3


root@master-01:~# kubectl get pods --all-namespaces -o wide
NAMESPACE      NAME                                READY   STATUS    RESTARTS   AGE     IP              NODE        NOMINATED NODE   READINESS GATES
kube-flannel   kube-flannel-ds-2g4td               1/1     Running   0          2m50s   192.168.10.18   worker-03   <none>           <none>
kube-flannel   kube-flannel-ds-9sc8t               1/1     Running   0          7m49s   192.168.10.7    worker-01   <none>           <none>
kube-flannel   kube-flannel-ds-ncz4c               1/1     Running   0          13m     192.168.10.16   master-01   <none>           <none>
kube-flannel   kube-flannel-ds-rwl82               1/1     Running   0          4m55s   192.168.10.28   worker-02   <none>           <none>
kube-flannel   kube-flannel-ds-xj87c               1/1     Running   0          74s     192.168.10.14   worker-04   <none>           <none>
kube-system    coredns-5d78c9869d-5jvjb            1/1     Running   0          16m     10.244.0.2      master-01   <none>           <none>
kube-system    coredns-5d78c9869d-7rsfw            1/1     Running   0          16m     10.244.0.3      master-01   <none>           <none>
kube-system    etcd-master-01                      1/1     Running   0          16m     192.168.10.16   master-01   <none>           <none>
kube-system    kube-apiserver-master-01            1/1     Running   0          16m     192.168.10.16   master-01   <none>           <none>
kube-system    kube-controller-manager-master-01   1/1     Running   0          16m     192.168.10.16   master-01   <none>           <none>
kube-system    kube-proxy-bmz4g                    1/1     Running   0          4m55s   192.168.10.28   worker-02   <none>           <none>
kube-system    kube-proxy-dw4s5                    1/1     Running   0          2m50s   192.168.10.18   worker-03   <none>           <none>
kube-system    kube-proxy-g8qvx                    1/1     Running   0          7m49s   192.168.10.7    worker-01   <none>           <none>
kube-system    kube-proxy-gxlqg                    1/1     Running   0          16m     192.168.10.16   master-01   <none>           <none>
kube-system    kube-proxy-t5lwh                    1/1     Running   0          74s     192.168.10.14   worker-04   <none>           <none>
kube-system    kube-scheduler-master-01            1/1     Running   0          16m     192.168.10.16   master-01   <none>           <none>

```

Убедились, что **все 5 нод (1 Master + 4 Worker) готовы к работе**

20. Удалил используемые ресурсы
