### Задание 1

**Выполните действия, приложите файлы с плейбуками и вывод выполнения.**

Напишите три плейбука. При написании рекомендуем использовать текстовый редактор с подсветкой синтаксиса YAML.

Плейбуки должны: 

1. Скачать какой-либо архив, создать папку для распаковки и распаковать скаченный архив. Например, можете использовать [официальный сайт](https://kafka.apache.org/downloads) и зеркало Apache Kafka. При этом можно скачать как исходный код, так и бинарные файлы, запакованные в архив — в нашем задании не принципиально.
2. Установить пакет tuned из стандартного репозитория вашей ОС. Запустить его, как демон — конфигурационный файл systemd появится автоматически при установке. Добавить tuned в автозагрузку.
3. Изменить приветствие системы (motd) при входе на любое другое. Пожалуйста, в этом задании используйте переменную для задания приветствия. Переменную можно задавать любым удобным способом.

### Решение 1
1. download_and_extract.yml

```
---
- name: Download and extract archive
  hosts: all
  become: yes
  tasks:
    - name: Create directory for extraction
      file:
        path: /opt/kafka
        state: directory

    - name: Download Kafka binary archive
      get_url:
        url: https://dlcdn.apache.org/kafka/3.9.0/kafka-3.9.0-src.tgz
        dest: /tmp/kafka.tgz

    - name: Extract the downloaded archive
      unarchive:
        src: /tmp/kafka.tgz
        dest: /opt/kafka
        remote_src: yes


```

2. install_tuned.yml

```
---
- name: Install and start tuned service
  hosts: all
  become: yes
  tasks:
    - name: Install tuned package
      apt:
        name: tuned
        state: present
      when: ansible_os_family == "Debian"

    - name: Install tuned package (for RedHat)
      yum:
        name: tuned
        state: present
      when: ansible_os_family == "RedHat"

    - name: Start and enable tuned service
      service:
        name: tuned
        state: started
        enabled: yes

```


3. change_motd.yml
   
```
---
- name: Change MOTD with a variable
  hosts: all
  become: yes
  vars:
    custom_motd: "Welcome to our server! Have a great day!"

  tasks:
    - name: Update MOTD file
      copy:
        content: "{{ custom_motd }}"
        dest: /etc/motd

```
![alt text](https://github.com/Dun9Dev/8.02HW/blob/main/img/Screenshot_20250205_002128.png)



### Задание 2

**Выполните действия, приложите файлы с модифицированным плейбуком и вывод выполнения.** 

Модифицируйте плейбук из пункта 3, задания 1. В качестве приветствия он должен установить IP-адрес и hostname управляемого хоста, пожелание хорошего дня системному администратору. 


modify_motd_with_host_details.yml
   
```
---
- name: Change MOTD with host details
  hosts: local
  become: yes
  tasks:
    - name: Gather facts
      setup:

    - name: Update MOTD file with host details
      copy:
        content: |
          Hostname: {{ ansible_hostname }}
          IP Address: {{ ansible_default_ipv4.address }}
          Have a great day, SysAdmin!
        dest: /etc/motd

```

![alt text](https://github.com/Dun9Dev/8.02HW/blob/main/img/Screenshot_20250205_005242.png)


### Задание 3

**Выполните действия, приложите архив с ролью и вывод выполнения.**

Ознакомьтесь со статьёй [«Ansible - это вам не bash»](https://habr.com/ru/post/494738/), сделайте соответствующие выводы и не используйте модули **shell** или **command** при выполнении задания.

Создайте плейбук, который будет включать в себя одну, созданную вами роль. Роль должна:

1. Установить веб-сервер Apache на управляемые хосты.
2. Сконфигурировать файл index.html c выводом характеристик каждого компьютера как веб-страницу по умолчанию для Apache. Необходимо включить CPU, RAM, величину первого HDD, IP-адрес.
Используйте [Ansible facts](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_vars_facts.html) и [jinja2-template](https://linuxways.net/centos/how-to-use-the-jinja2-template-in-ansible/). Необходимо реализовать handler: перезапуск Apache только в случае изменения файла конфигурации Apache.
4. Открыть порт 80, если необходимо, запустить сервер и добавить его в автозагрузку.
5. Сделать проверку доступности веб-сайта (ответ 200, модуль uri).

В качестве решения:
- предоставьте плейбук, использующий роль;
- разместите архив созданной роли у себя на Google диске и приложите ссылку на роль в своём решении;
- предоставьте скриншоты выполнения плейбука;
- предоставьте скриншот браузера, отображающего сконфигурированный index.html в качестве сайта.
