### Kubernetes - Эта инструкция предназначена для использования на Linux-серверах

## 1. Установка Docker
Для начала работы с k8s, необходимо установить Docker на каждом сервере в кластере:
```bash
$ sudo apt-get update && sudo apt-get install docker.io -y  # Установка Docker на linux тачке
```

## 2. Установка Kubernetes
Для установки Kubernetes необходимо добавить репозиторий и установить пакеты kubeadm, kubectl и kubelet:
```bash
$ sudo apt-get update && sudo apt-get install -y apt-transport-https curl             # Установка курла
$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -  # Добавление ключа
$ cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list                        # Добавление репозитория
deb https://apt.kubernetes.io/ kubernetes-xenial main                                 # Вводим в окне
EOF                                                                                   # Вводим в окне
$ sudo apt-get update                                                                 # Обновляем репозитории
$ sudo apt-get install -y kubelet kubeadm kubectl                                     # Устанавливаем kubelet kubeadm kubectl
```

## 3. Инициализация мастера Kubernetes
После установки kubeadm мы можем использовать его для инициализации мастера Kubernetes:
```bash
$ sudo kubeadm init  # Создаём новый кластер Kubernetes на мастер-узле
```
В процессе выполнения команды будут созданы сертификаты, настроены конфигурационные файлы и запущен API-сервер Kubernetes.
Вы можете скопировать команду вывода, которая появится в консоли, и выполнить ее на рабочих узлах, чтобы присоединить их к кластеру.

## 4. Настройка кластера
После инициализации кластера вам необходимо настроить его, чтобы он был готов к использованию. Следуйте инструкциям, которые вы увидите в выводе команды kubeadm init. Например:
```bash
$ mkdir -p $HOME/.kube                                      # Создаём папку для хранения конфигов кубера
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config  # Переносим конфиги в нашу домашнюю директорию для удобства
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config           # Настраиваем права
```

## 5. Добавление рабочих узлов
Когда кластер настроен, вы можете добавить к нему рабочие узлы
Для этого вам нужно выполнить команду, которая появится в выводе команды ```bash $ kubeadm init``` на каждом из рабочих узлов:
```bash
$ sudo kubeadm join <MASTER_NODE_IP>:<MASTER_NODE_PORT> --token <TOKEN> --discovery-token-ca-cert-hash <DISCOVERY_TOKEN_CA_CERT_HASH>  # Добавляем воркер ноды к нашей мастер ноде
```
Вы можете найти необходимые данные для подстановки в эту команду, выполнив следующую команду на мастер-узле:
```bash
$ kubeadm token create --print-join-command  # Создаём токен на MASTER ноде и показываем все данные для добавления воркеров к мастер
```

## 6. Проверка кластера
После добавления рабочих узлов вы можете проверить состояние вашего кластера Kubernetes с помощью команды:
```bash
$ kubectl get nodes  # Посмотреть все ноды в кластере и их состояние
```

## 7. Установка сетевого плагина
Последний шаг в создании кластера Kubernetes - установка сетевого плагина. Сетевой плагин - это программное обеспечение, которое обеспечивает связь между различными контейнерами в кластере. Вы можете установить один из нескольких популярных сетевых плагинов, таких как Calico, Flannel или Weave Net. В этом примере мы установим сетевой плагин Calico:
```bash
$ kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml  # Устанавливаем сетевой плагин Calico
```

## 8. Использование кластера
Теперь ваш кластер Kubernetes готов к использованию. Вы можете разворачивать и управлять приложениями в кластере, используя команды kubectl. Например, чтобы запустить приложение, вы можете выполнить команду:
```bash
$ kubectl run my-app --image=my-image --replicas=3  # Запускаем три экземпляра контейнера, основанных на образе my-image
```

## 9. Установка Kubernetes Dashboard (необязательно)
Kubernetes Dashboard - это веб-интерфейс, который позволяет визуализировать и управлять кластером Kubernetes. Если вы хотите использовать Kubernetes Dashboard, вы можете установить его следующим образом:
```bash
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta8/aio/deploy/recommended.yaml
```
После установки Kubernetes Dashboard, вы можете запустить его, выполним команду:
```bash
$ kubectl proxy
```
Затем откройте браузер и перейдите по адресу ```http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/```

## 10. Установка Helm (необязательно)
Helm - это пакетный менеджер для Kubernetes, который позволяет управлять и устанавливать приложения в кластере. Если вы хотите использовать Helm, вы можете установить его следующим образом:
```bash
$ curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
```

## 11. Создание namespace (необязательно)
Namespace - это способ разделения кластера Kubernetes на логические части. Если вы хотите создать новый namespace, вы можете выполнить команду:
```bash
$ kubectl create namespace my-namespace
```

## 12. Установка мониторинга (НЕ ПРОВЕРЕНО)
Для мониторинга состояния кластера Kubernetes можно использовать различные инструменты, например, Prometheus, Grafana и Alertmanager.
Для установки мониторинга с помощью Prometheus и Grafana, вы можете выполнить следующие команды:
```bash
$ kubectl create namespace monitoring
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.0.0/deploy/static/provider/baremetal/deploy.yaml -n monitoring
$ kubectl apply -f https://raw.githubusercontent.com/prometheus-community/helm-charts/main/charts/kube-prometheus-stack/values.yaml -n monitoring
$ helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
$ helm repo update
$ helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring
$ kubectl apply -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/main/example/rbac/prometheus-operator-namespace.yaml -n monitoring
$ kubectl apply -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/main/example/rbac/prometheus-operator.yaml -n monitoring
$ helm install prometheus-operator prometheus-community/prometheus-operator -n monitoring
$ kubectl apply -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/main/example/prometheus-operator/prometheus.yaml -n monitoring
```
Эти команды создают namespace monitoring, устанавливают контроллер Ingress, устанавливают Prometheus и Grafana с помощью Helm, а также устанавливают Prometheus Operator
После установки мониторинга, вы можете открыть Grafana в браузере, перейдя по адресу http://<IP-адрес-кластера>:3000 и войдя с помощью учетных данных, указанных в файле values.yaml при установке kube-prometheus-stack

