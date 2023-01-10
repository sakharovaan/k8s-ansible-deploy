# ansible-kubernetes

## Инициализация среды

1. Перед началом установить python 3.8
2. Скопировать ansible.example.cfg в ansible.cfg
```
sudo pip3 install pipenv
pipenv sync
pipenv shell
```

## Использование

Применить плейбуки ко всем нодам

`ansible-playbook site.yml -i inventory/name -vD`

Применить плейбуки к одной ноде

`ansible-playbook site.yml -i inventory/name -l kube-worker-3 -vD`

Запустить в тестовом режиме (aka dry run, но не все модули поддерживают и могут быть ошибки)

`ansible-playbook site.yml -i inventory/name -vDC`

Можно запускать ansible без плейбуков, например получить сведения о localhost в формате json:

`ansible localhost -m k8s_facts -a "kind=Node" -i inventory/name -vD`

Запустить команду на всех нодах:

`ansible all -m shell -a "uname -a" -i inventory/name -vD`

Подробнее https://docs.ansible.com/ansible/latest/user_guide/intro_adhoc.html#why-use-ad-hoc-commands

### Добавление ноды в kubernetes

1. Добавить файл в inventory/hosts с нужными параметрами и добавить хост в нужные группы в groups
2. Запустить `ansible-playbook site.yml -i inventory/name -l kube-test -vD`
3. Запустить `ansible-playbook site.yml -i inventory/name -l kube-test -t kube_node_join -vD` -- при этом запустится обновление пакетов и перезагрузка ноды

Для добавления первого мастера `-t kube_master_init`
Для добавления второго и последующего мастера `-t kube_master_join`

### Обновление kubernetes

1. Убедиться, что в roles/kubernetes/defaults/main.yml (или host/group vars) стоит в kubernetes_version и kubernetes_package_version текущая версия кластера и deb-пакета, а в kubernetes_upgrade_version и kubernetes_upgrade_package_version та версия, до которой нужно обновляться
2. Для обновления кластера на первой мастер-ноде: `ansible-playbook site.yml -l kube-main-master -t kube_master_upgrade_main -vD`
3. Для обновления остальных мастер-нод: `ansible-playbook site.yml -l kube-sec-master -t kube_master_upgrade_secondary -vD`
4. Для обновления рабочих нод: `ansible-playbook site.yml -l kube-worker -t kube_node_upgrade -vD`
5. После обновления версию в переменных kubernetes_(package_)version сделать такой же как в kubernetes_upgrade_(package_)version 
