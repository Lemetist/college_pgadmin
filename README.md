# Настройка Ansible для управления клиентскими машинами

## Подготовка сервера

### 1. Установка Ansible

```bash
apt-get install ansible
```

### 2. Добавление управляемых хостов

Редактируем файл `/etc/ansible/hosts` и добавляем следующее содержимое:

```
[all:vars]
ansible_user=root

[local]
10.10.3.77
```

### 3. Создание SSH-ключа

```bash
ssh-keygen -t ed25519 -f ~/.ssh/manager
```

Парольную фразу можно оставить пустой.

### 4. Добавление ключа в агент SSH

```bash
eval `ssh-agent`
ssh-add ~/.ssh/manager
```

### 5. Создание каталога для плейбуков

```bash
mkdir -p /etc/ansible/playbooks
```

## Подготовка клиента

### 1. Установка необходимых модулей

```bash
apt-get install python python-module-yaml python-module-jinja2 python-modules-json python-modules-distutils
```

### 2. Включение и запуск службы SSH

Для SystemV:

```bash
chkconfig sshd on
service sshd start
```

Для SystemD:

```bash
systemctl enable sshd.service
systemctl start sshd.service
```

### 3. Размещение публичного SSH-ключа

Копируем публичный ключ на клиент:

```bash
ssh-copy-id -i ~/.ssh/manager.pub root@10.10.3.77
```

Или вручную добавляем содержимое `manager.pub` в файл `/root/.ssh/authorized_keys`.

### 4. Проверка подключения

```bash
ssh root@10.10.3.77
```

## Проверка доступности

Проверяем связь с клиентом через Ansible:

```bash
ansible -m ping local
```

Пример успешного ответа:

```json
10.10.3.77 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

## Полезные плейбуки

### 1. Прописывание репозитория

Файл: `/etc/ansible/playbooks/repo.yml`

```yaml
- hosts: local
  remote_user: root
  tasks:
  - name: Remove all repositories
    shell: apt-repo rm all
  - name: Add official mirror
    shell: apt-repo add http://10.10.3.77/repo/p8
  - name: Add official mirror with arepo
    shell: apt-repo add 'rpm http://10.10.3.77/repo/p8 x86_64-i586 classic'
  - name: Add extra repository
    shell: apt-repo add 'rpm http://10.10.3.77/repo/extra x86_64 extra'
```

Запуск:

```bash
ansible-playbook /etc/ansible/playbooks/repo.yml
```

### 2. Установка пакета

Файл: `/etc/ansible/playbooks/install-ifcplugin.yml`

```yaml
- hosts: local
  remote_user: root
  tasks:
  - name: Update cache and install ifcplugin
    apt:
      name: ifcplugin
      state: present
      update_cache: yes
```

Запуск:

```bash
ansible-playbook /etc/ansible/playbooks/install-ifcplugin.yml
```

### 3. Обновление системы

Файл: `/etc/ansible/playbooks/update_system.yml`

```yaml
- hosts: local
  remote_user: root
  gather_facts: no
  tasks:
  - name: Update cache
    apt:
      update_cache: true
  - name: Upgrade system
    apt:
      upgrade: dist
  - name: Upgrade kernel
    shell: apt-get install --only-upgrade linux-image-generic
  - name: Clean package cache
    apt:
      autoclean: yes
```

Запуск:

```bash
ansible-playbook /etc/ansible/playbooks/update_system.yml
```

