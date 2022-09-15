# Учим vagrant и ansible
Развертывает несколько виртуальных машин, и на одной из них поднимает контейнеры.<br>
### Результат:
1. Запущены три виртуальных машины
2. Софт на данных машинах обновлен
3. Запущены приложения в докер-контейнерах на хосте `social_bots`

### Порядок установки
1. Скачать проект
2. Создать виртуальное окружение
```commandline
python -m venv venv && . ./venv/bin/activate
```
3. Установить зависимости 
```commandline
pip install -r requirements.txt
```

### Установка и запуск виртуальных машин (VM) на хосте
1. Установить vagrant
```commandline
sudo apt update && sudo apt install vagrant && vagrant --version
```

2. Скачать образ VM (из-за санкций приходится скачивать на хост)
```commandline
wget -O "box/ubuntu-jammy64" https://app.vagrantup.com/ubuntu/boxes/jammy64/versions/20220910.0.0/providers/virtualbox.box
```

3. Добавить образ `ubuntu-jammy64` в vagrant
```commandline
vagrant box add box/ubuntu-jammy64 --name ubuntu-jammy64
```

4. Создать `Vagrantfile`, если его нет в директории `playbooks`
```commandline
vagrant init ubuntu-jammy64
```

5. Поднимаем VM, указанные в `Vagrantfile`
```commandline
vagrant up
```

6. Проверить статус поднятых VM
```commandline
vagrant status
```

7. Проверить ssh-config и доступ по ssh (выход из VM - `exit`)
```commandline
vagrant ssh-config && vagrant ssh vagrant1
```

### Ansible
1. Установить ansible
```commandline
pip install ansible
```
2. Создать файл `host.ini` (данные взять из `vagrant ssh-config`)
```ini
social_bots ansible_ssh_host=127.0.0.1 ansible_ssh_port=2222
rdb ansible_ssh_host=127.0.0.1 ansible_ssh_port=2200
vagrant1 ansible_ssh_host=127.0.0.1 ansible_ssh_port=2201

[vagrant]
vagrant1

[bots]
social_bots

[db]
rdb

```
3. Создать файл ansible.cfg
```ini
[defaults]
inventory = inventory/hosts.ini
remote_user = vagrant
private_key_file = ~/.vagrant.d/insecure_private_key
host_key_checking = False
```
4. Проверить доступ через ansible
```commandline
ansible vagrant1 -m ping && ansible rdb -m ping && ansible social_bots -m ping 
```
5. В `playbooks/roles/social_bots/files/env` разместить файлы `tg_secrets.env` и `vk_secrets.env`
6. Запустить `playbook.yml`
```commandline
ansible-playbook playbook.yml
```

