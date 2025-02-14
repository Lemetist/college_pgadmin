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



