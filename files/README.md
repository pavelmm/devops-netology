# 8.2 описание Playbook по заданию


## Описание Play 

### Install Java
 - загрузка установосного пакета
 - распаковка пакета


# Домашнее задание к занятию "08.02 Работа с Playbook"

## Подготовка к выполнению

1. Создайте свой собственный (или используйте старый) публичный репозиторий на github с произвольным именем.
2. Скачайте [playbook](./playbook/) из репозитория с домашним заданием и перенесите его в свой репозиторий.
3. Подготовьте хосты в соответствии с группами из предподготовленного playbook.

## Основная часть

1. Приготовьте свой собственный inventory файл `prod.yml`.
```
[root@localhost es2]# cat prod.yml 
---
- name: Install Vector for server
  hosts: all
  become: yes

  tasks:
    
  - name: Install Vector
    yum: name=https://packages.timber.io/vector/0.21.2/vector-0.21.2-1.x86_64.rpm

```


2. Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает [vector](https://vector.dev). `Готово`
3. При создании tasks рекомендую использовать модули: `get_url`, `template`, `unarchive`, `file`.
4. Tasks должны: скачать нужной версии дистрибутив, выполнить распаковку в выбранную директорию, установить vector. `Готово`
5. Запустите `ansible-lint site.yml` и исправьте ошибки, если они есть.
```
[root@localhost es2]# ansible-lint prod.yml 
 [WARNING] Ansible is in a world writable directory (/root/es2), ignoring it as an ansible.cfg source.
[ANSIBLE0002] Trailing whitespace
prod.yml:7
```
6. Попробуйте запустить playbook на этом окружении с флагом `--check`.
```
[root@localhost es2]# ansible-playbook prod.yml -K --check
BECOME password: 

PLAY [Install Vector for server] ******************************************************

TASK [Gathering Facts] ****************************************************************
ok: [linux1]

TASK [Install Vector] *****************************************************************
ok: [linux1]

PLAY RECAP ****************************************************************************
linux1                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
7. Запустите playbook на `prod.yml` окружении с флагом `--diff`. Убедитесь, что изменения на системе произведены.
```
[root@localhost es2]# ansible-playbook prod.yml -K --diff
BECOME password: 

PLAY [Install Vector for server] ******************************************************

TASK [Gathering Facts] ****************************************************************
ok: [linux1]

TASK [Install Vector] *****************************************************************
ok: [linux1]

PLAY RECAP ****************************************************************************
linux1                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```

8. Повторно запустите playbook с флагом `--diff` и убедитесь, что playbook идемпотентен.
```
[root@localhost es2]# ansible-playbook prod.yml -K --diff
BECOME password: 

PLAY [Install Vector for server] ******************************************************

TASK [Gathering Facts] ****************************************************************
ok: [linux1]

TASK [Install Vector] *****************************************************************
ok: [linux1]

PLAY RECAP ****************************************************************************
linux1                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```
9. Подготовьте README.md файл по своему playbook. В нём должно быть описано: что делает playbook, какие у него есть параметры и теги. `Готово`
10. Готовый playbook выложите в свой репозиторий, поставьте тег `08-ansible-02-playbook` на фиксирующий коммит, в ответ предоставьте ссылку на него.
https://github.com/pavelmm/devops-netology/blob/main/files/README.md

---

### Как оформить ДЗ?

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
