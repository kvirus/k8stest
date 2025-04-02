# [OPS-948]
## Создание кластера Kubernetes, развертывание docker-registry, nginx
### 1. Создаем дополнительнительную ВМ:
а. Создаем дополнительную ВМ с двумя сетевыми картами (для внешнего доступа и изолированную карту).
б. Настраиваем как остальные ВМ.

 ### 2. Роль установки и настройки Kubernetes:
Название роли: <code>k8s</code>
Роль осуществляет преднастройку ОС для кластера Kubernetes:<br>
 1. Предустановки:<br>
 - Отключение Swap<br>
 - Удаление Swap из fstab<br>
 - Подключение модулей и параметров ядра<br>
 2. Установка модулей на ОС Centos и Ubuntu.<br>
 3. Инициализация кластера.<br>
 4. Установка лейблов для нод. Front и Back.<br>

 Теги роли:
```
  k8s - Общий тег роли
  k8s-setup - Предустановки ОС
  k8s-centos-install - Установка компонентов на ОС Centos
  k8s-ubuntu-install - Установка компонентов на ОС Ubuntu
  k8s-cluster-init - Инициализация кластера
```
### 3. Роль развертывания Nginx:
Название роли <code>k8s-nginx</code>
Роль осуществляет:
 - Копирование файлов развертывания на мастер-ноду.
 - Копирование ключей (через kubernetes secret).
 - Копирование настроек доступа для Registry (через configmap).
 - Деплой Nginx на ноду Front.
 - Установка сервиса, для доступа к Nginx.
 Теги роли:
```
    k8s-nginx - Копирование файлов деплоя, конфигурации и деплой Nginx
```
### 4. Роль развертывания Registry:
Название роли <code>k8s-registry</code>
Роль осуществляет:
 - Копирование файлов развертывания на мастер-ноду.
 - Разворачивание docker-registry как StatefulSet на ноду back.
 Теги роли:
```
  k8s-registry - Развертывание docker-registry как StatefulSet
```






#### 2. Настройка :
*настройки проводим на всех 3х узлах*<br><br>
а. Отключаем swap:<br>
```
Закоменчиваем в: nano /etc/fstab
Отключаем: sudo swapoff -a
```
б. Загружаем модули ядра:<br>

Добавим в файл:

```
nano /etc/modules-load.d/k8s.conf
br_netfilter
overlay
```
Активируем:

```
sudo modprobe br_netfilter
sudo modprobe overlay
```
в. Настроим сетевые параметры:

```
nano /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
```
И применим их:

```
sudo sysctl --system
Проверка:
sudo lsmod | egrep "br_netfilter|overlay"
```
#### 3. Установка k8s:
Устанавливаем необходимые пакеты:
```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl containerd

Холдим пакеты:
sudo apt-mark hold kubelet kubeadm kubectl
```
#### 4. Инициализация кластера (на Master ноде):
Инициализируем кластер:

```
kubeadm init  --pod-network-cidr=10.244.0.0/16
```
Будет выдан ключ подключения кластера (для worker nodes):

```
kubeadm join 10.0.15.32:6443 --token 8x9vcz.yljvma21e2bsro66 \
        --discovery-token-ca-cert-hash sha256:e5fa2979d74e05d74c0547307f67fa9baf25faba97ec023c8978f8bc89d820e3
```
Данная строка подключения будет нужна для подключения worker nodes

#### 5. Подключение worker nodes:

Вводим на нодах строку подключения выше и ноды подключатся.

#### 6. Устанавливаем лейблы:
Устанавливаем лейблы:

```
kubectl label nodes node1 front
kubectl label nodes node2 back
```
#### 7. Разворачиваем docker-registry как StatefulSet (создаем манифесты):

```
registry-namespace.yaml  - Создаем новый Namespace
registry-deploy.yml - Деплой registry
registry-service.yml - делаем сервис для доступа к registry (как ClusterIP)
nginx-configmap.yml - Создаем конфигурацию для Nginx
nginx-deploy.yml - Деплой для Nginx
nginx-service.yml - Создаем сервис для Nginx
```

#### 8. Применяем созданные манифасты:

```
kubectl apply -f registry-namespace.yaml
kubectl apply -f registry-deploy.yml
kubectl apply -f registry-service.yml
kubectl apply -f nginx-configmap.yml
kubectl apply -f nginx-deploy.yml
kubectl apply -f nginx-service.yml
```
Проверяем работу Docker-registry:

```
curl -X GET http://http://registry:5000/v2/
В ответ получим пустой json {}
```