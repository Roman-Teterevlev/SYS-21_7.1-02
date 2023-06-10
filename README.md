# SYS-21_7.1-02
Ansible.Часть 2
## Задание 1
Выполните действия, приложите файлы с плейбуками и вывод выполнения.

Напишите три плейбука. При написании рекомендуем использовать текстовый редактор с подсветкой синтаксиса YAML.

Плейбуки должны:

1. Скачать какой-либо архив, создать папку для распаковки и распаковать скаченный архив. Например, можете использовать официальный сайт и зеркало Apache Kafka. При этом можно скачать как исходный код, так и бинарные файлы, запакованные в архив — в нашем задании не принципиально.
2. Установить пакет tuned из стандартного репозитория вашей ОС. Запустить его, как демон — конфигурационный файл systemd появится автоматически при установке. Добавить tuned в автозагрузку.
3. Изменить приветствие системы (motd) при входе на любое другое. Пожалуйста, в этом задании используйте переменную для задания приветствия. Переменную можно задавать любым удобным способом.

### Ответ:
1.
```
---
- name: test1
  hosts: servers
  become: yes

  tasks:

  - name: Download and Unarchive
    unarchive:
      src: https://downloads.apache.org/kafka/3.4.1/kafka-3.4.1-src.tgz
      dest: /usr/local
      remote_src: yes
...
```
<img width="960" alt="7-01 1_1" src="https://github.com/Roman-Teterevlev/SYS-21_7.1-02/assets/132853752/195ab8f1-0798-4d41-ab45-bc8d674fc6c0">
<img width="960" alt="7-01 1_2" src="https://github.com/Roman-Teterevlev/SYS-21_7.1-02/assets/132853752/691b0e09-3405-4531-92ee-90215cb42e28">
<img width="960" alt="7-01 1_3" src="https://github.com/Roman-Teterevlev/SYS-21_7.1-02/assets/132853752/940a92a1-cb8a-4bbc-9e84-a03e0f4d3076">

2.
```
---
- name: test2
  hosts: servers
  become: true

  tasks:

  - name: Install the tuned
    apt:
      name: tuned
      state: present

  - name: Start the tuned
    systemd:
      name: tuned
      state: started
      enabled: yes
    become: yes
    become_user: root
...
```
<img width="960" alt="7-01 1_4" src="https://github.com/Roman-Teterevlev/SYS-21_7.1-02/assets/132853752/5a0f6970-bfec-492c-9c08-432b581e7ab4">
<img width="960" alt="7-01 1_5" src="https://github.com/Roman-Teterevlev/SYS-21_7.1-02/assets/132853752/a09ca835-9acc-4b28-91ae-6ed543283cfd">

3.
```
---
- name: test3
  hosts: servers
  become: true

  vars:
    path: /etc/update-motd.d/

  tasks:

  - name: Entrance change
    file:
      path: "{{path}}"
      mode: u=rw,g=rw,o=rw
      recurse: yes
...
```
![7-01 1_6](https://github.com/Roman-Teterevlev/SYS-21_7.1-02/assets/132853752/b4e245c1-8ee8-4e63-9cbe-c7bb90970490)
![7-01 1_7](https://github.com/Roman-Teterevlev/SYS-21_7.1-02/assets/132853752/ae9be71e-3ca5-4d96-be18-d4223215f8c8)
![7-01 1_8](https://github.com/Roman-Teterevlev/SYS-21_7.1-02/assets/132853752/0e00974d-88b2-45d4-b1fc-db9cfa18e6b8)

## Задание 2
Выполните действия, приложите файлы с модифицированным плейбуком и вывод выполнения.

Модифицируйте плейбук из пункта 3, задания 1. В качестве приветствия он должен установить IP-адрес и hostname управляемого хоста, пожелание хорошего дня системному администратору.

### Ответ:
``` vi motd.j2 ```

``` Hello, and welcome to "{{ ansible_facts['fqdn'] }}" ```

```
---
- name: test3
  hosts: servers
  become: true

  vars:
    path: /etc/update-motd.d/

  tasks:

  - name: Entrance change
    file:
      path: "{{path}}"
      mode: u=rw,g=rw,o=rw
      recurse: yes

  - name: Add the message
    template:
      src: motd.j2
      dest: /etc/motd
      mode: '644'
...
```
![7-01 1_9](https://github.com/Roman-Teterevlev/SYS-21_7.1-02/assets/132853752/92249243-d09b-42a1-a5c4-690e79e4e09a)
![7-01 1_10](https://github.com/Roman-Teterevlev/SYS-21_7.1-02/assets/132853752/b3b6993d-4c02-491f-9f2f-209477dd6b47)

## Задание 3
Выполните действия, приложите архив с ролью и вывод выполнения.

Ознакомьтесь со статьёй «Ansible - это вам не bash», сделайте соответствующие выводы и не используйте модули shell или command при выполнении задания.

Создайте плейбук, который будет включать в себя одну, созданную вами роль. Роль должна:

1. Установить веб-сервер Apache на управляемые хосты.
2. Сконфигурировать файл index.html c выводом характеристик каждого компьютера как веб-страницу по умолчанию для Apache. Необходимо включить CPU, RAM, величину первого HDD, IP-адрес. Используйте Ansible facts и jinja2-template
3. Открыть порт 80, если необходимо, запустить сервер и добавить его в автозагрузку.
4. Сделать проверку доступности веб-сайта (ответ 200, модуль uri).
В качестве решения:

- предоставьте плейбук, использующий роль;
- разместите архив созданной роли у себя на Google диске и приложите ссылку на роль в своём решении;
- предоставьте скриншоты выполнения плейбука;
- предоставьте скриншот браузера, отображающего сконфигурированный index.html в качестве сайта.

### Ответ:
``` ansible@v824685:~$ cat ansible/roles/apache/tasks/install.yaml ```
```
---
  - name: install apache web server
    apt:
      name:
      - apache2
      state: present
```
``` ansible@v824685:~$ cat ansible/roles/apache/tasks/service.yaml ```
```
---
  - name: start apache webserver
    service:
      name: apache2
      state: started
      enabled: true
```
``` ansible@v824685:~$ cat ansible/roles/apache/tasks/port.yaml ```
```
---
  - name: wait for port 80 to become open
    wait_for:
      port: 80
      delay: 10
```
``` ansible@v824685:~$ cat ansible/roles/apache/tasks/index.yaml ```
```
---
- name: add the index page
  template:
    src: "index.html.j2"
    dest: "/var/www/html/index.html"
    owner: root
    group: root
    mode: 0755
```
``` ansible@v824685:~$ cat ansible/roles/apache/tasks/connection.yaml ```
```
---
  - name: Check that you can connect (GET) to a page and it returns a status 200
    ansible.builtin.uri:
      url: "{{address}}"
    vars:
      address: "http://{{ ansible_facts.all_ipv4_addresses [0] }}"
```
``` ansible@v824685:~$ cat ansible/roles/apache/tasks/main.yml ```
```
---
# tasks file for apache
  - import_tasks: install.yaml
  - import_tasks: service.yaml
  - import_tasks: port.yaml
  - import_tasks: index.yaml
  - import_tasks: connection.yaml
```
``` ansible@v824685:~$ cat ansible/roles/apache/templates/index.html.j2 ```
```
<p>IP: {{ ansible_facts.all_ipv4_addresses [0] }}
<p>CPU: {{ ansible_facts.processor }}
<p>RAM_mb: {{ ansible_facts.memtotal_mb }}
<p>vda1_size: {{ ansible_facts['devices']['vda']['partitions']['vda1']['size'] }}
```
``` ansible@v824685:~/ansible$ cat test4.yaml ```
```
---
- name: test4
  hosts: servers
  become: true
  roles:
    - apache
```

