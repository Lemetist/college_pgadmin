Подготовка сервера
1. Устанавливаем ansible:

apt-get install ansible
2. Прописываем управляемые компьютеры в группу (local). В файл /etc/ansible/hosts требуется добавить (а в новых версиях Ansible желательно указать пользователя, под которым будет запущены команды на управляемых узлах (root)):

[all:vars]
ansible_user=root

[local]
10.10.3.77
3. Так как управление будет осуществляться через ssh по ключам, создаём ключ SSH:

ssh-keygen -t ed25519 -f ~/.ssh/manager
Пароль ключа можно оставить пустым.

4. Добавить ключ на сервере:

eval `ssh-agent`
ssh-add ~/.ssh/manager
5. Создадим каталог для пакетов рецептов (playbooks):

mkdir -p /etc/ansible/playbooks
Подготовка клиента
1. Устанавливаем необходимые модули:

apt-get install python python-module-yaml python-module-jinja2 python-modules-json python-modules-distutils
2. Включаем и запускаем службу sshd:

для SystemV:

chkconfig sshd on
service sshd start
для SystemD:

systemctl enable sshd.service
systemctl start sshd.service
3. Размещаем публичную часть созданного на сервере ключа пользователю root (в модуле Администратор или вручную добавить содержимое файла manager.pub в /etc/openssh/authorized_keys/root.

4. Проверим доступ по ключу с сервера:

ssh root@10.10.3.77
Проверка доступности
Используем модуль «ping»:

# ansible -m ping local
10.10.3.77 | SUCCESS => {
    "changed": false, 
    "failed": false, 
    "ping": "pong"
}
Полезные рецепты
Рецепты применяются командой:

ansible-playbook <имя файла>
Прописывание репозитория
Файл: /etc/ansible/playbooks/repo.yml

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
Примечание: Используется модуль shell и программа apt-repo.


Установка пакета
Файл: /etc/ansible/playbooks/install-ifcplugin.yml

- hosts: local
  remote_user: root
  tasks:
  - name: Update cache and install ifcplugin
    apt_rpm:
      name: ifcplugin
      state: present
      update_cache: yes
Обновление системы
С версии ansible-2.9.27-alt2 и ansible-core-2.14.2-alt1:

- hosts: local
  remote_user: root
  gather_facts: no
  tasks:
  - name: Update cache
    apt_rpm:
      update_cache: true
  - name: Upgrade system
    apt_rpm:
      dist_upgrade: true
  - name: Upgrade kernel
    apt_rpm:
      update_kernel: true
  - name: Clean package cache
    apt_rpm:
      clean: true
Или всё в одном:

- hosts: local
  remote_user: root
  gather_facts: no
  tasks:
  - name: Upgrade system
    apt_rpm:
      update_cache: true
      dist_upgrade: true
      update_kernel: true
      clean: true
