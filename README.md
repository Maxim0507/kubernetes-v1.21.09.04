## ***`kubernetes v1.21.09.04`***
текущий стабильный релиз

### `Системные требования`

Операционная система
- Ubuntu 20.04
- Centos 7.9

`Минимальные требования к нодам Kubernetes`
- CPU 10
- RAM 24G
- HDD 100G

## `Процесс установки`
>Необходим доступ к репозиторию с пакетами дистрибутива в любом виде, например через примонтированный диск с дистрибутивом.

>В этот репозиторий включены все артефакты необходимые для разворачивания кластера Kubernetes кроме пакетов операционной системы и образов

- Подключаемся к машине, с которой будем производить установку под пользователем с правами sudo

- Создаем папки

```
sudo mkdir -p /opt/distr

sudo chown -R $(whoami) /opt/distr
```
- копируем в папку /opt/distr дистрибутив QB

- распаковываем архив

```
1. cd /opt/distr
    2. tar xfvz QB-*.tar.gz
        3. mv QB-*/ QB

```
- генерируем доступ по ключу к машинам на которые будет устанавливаться kubernetes
На машине с которой будем устанавливаться выполняем команды
```
ssh-agent bash
ssh-keygen -t rsa -f ~/.ssh/id_rsa -P ""
```
- Выполняем команду указав ip адрес и user - имя пользователя сервера на который будет происходить установка
на вопрос вводим yes и жмешь enter, вводим пароль
```
ssh-copy-id -i $HOME/.ssh/id_rsa.pub user@ip
```
- копируем пример инвентори

```
cd  /opt/distr/QB

cp -r inventory/ethalon inventory/distr
```
- редактируем файл, указываем полное имя ноды (FQDN) кластера вместо node1
в 4-х местах и ip адрес ноды в параметре ansible_host
```
vim /opt/distr/QB/inventory/distr/inventory.ini
```
- Подкладываем корневой сертификат реестра образов (ca.crt) 
в папку /opt/distr/QB/roles/add-certs/files/ca/ Назвав его по имени хоста  hostname.crt

- запускаем скрипт установки ansible
```
cd /opt/distr/QB/scripts
./install-ansible.sh
```
- запускаем скрипт замены имени хоста
```
./rep.sh
```
- запустить файловый сервер
```
cd /opt/distr/QB
nohup ./goFil 10867 > /dev/null 2>&1&
```

- запускаем плейбук preinstall, потребуется ввести пароль пользователя 
```
cd /opt/distr/QB

ansible-playbook -i inventory/distr/inventory.ini preinstall.yml -K 
```

- если прошло без ошибок (failed=0 в результате выполнения), запускаем плейбук установки

```
cd /opt/distr/QB/kubespray
ansible-playbook -i ../inventory/distr/inventory.ini --become --become-user=root cluster.yml
```

- после успешной установки, устанавливаем  qkibana, qfluentbit, qelasticsearch, qkafka

```
sudo kubectl apply -f /opt/distr/QB/yaml
```
- подключаемся к мастер ноде и зачитываем токен для подключения в дашборд  

- админ со всеми правами на кластер
```
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep "^admin-user"| awk '{print $1}')
```

## `Полезные ссылки`

[rebrain-devops-task1](https://gitlab.rebrainme.com/devops_users_repos/4621/rebrain-devops-task1)

[Гайд по Markdown](https://docs.github.com/ru/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax)

[devops](https://rebrainme.com/devops/)

## `Задачи`
- [х] Обновить kubernetes
- [х] Проверить сертификаты
- [ ] Проверить доступа в дашборд

