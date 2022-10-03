# Домашнее задание к занятию "13.4 инструменты для упрощения написания конфигурационных файлов. Helm и Jsonnet"

## Предварительная подготовка

В задании используется раннее настроенный кластер из трёх нод: control plane и две worker ноды.

## Задание 1: подготовка helm чарта для приложения

Создаём шаблон чарта при помощи команды:

```bash
[node5 ~]$ helm create my-chart
```

Затем, находясь в папке с чартом, были проделаны действия по адаптации deploy, sts и так далее, но для использования в виде шаблонов. Также был добавлен шаблон для развёртывания PVC. После подготовки чарта, его нужно проверить:

```bash
[node5 13.4]$ helm lint my-chart/
==> Linting my-chart/
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, 0 chart(s) failed
```
На этом этапе можно сгенерировать, то, что выдаётся при сборке шаблона:

```bash
helm template netology-demo my-chart/ >netology-demo.yaml
```


## Задание 2: запустить 2 версии в разных неймспейсах

Для того, чтобы запустить чарты в разных неймспейсах, можно воспользоваться ключами в командной строке и получить нужный результат:

```bash
[node5 13.4]$ helm upgrade v1 ./my-chart/ --install --namespace app-v1
Release "v1" does not exist. Installing it now.
NAME: v1
LAST DEPLOYED: Sun Oct  2 15:22:25 2022
NAMESPACE: app-v1
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:

[node5 13.4]$ helm list --all-namespaces -d
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS        CHART            APP VERSION
v1      app-v1          1               2022-10-02 15:22:25.583786681 +0000 UTC deployed      my-chart-0.1.0   1.16.0     

```
Вводим команды:

```bash
[node5 13.4]$ helm upgrade v2 ./my-chart/ --install --create-namespace --namespace app-v1
[node5 13.4]$ helm upgrade v3 ./my-chart/ --install --create-namespace --namespace app-v2
```
Смотрим в каком неймспейсе какой чарт и какой версии находится:

```bash
[node5 13.4]$ helm list --all-namespaces -d
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS        CHART            APP VERSION
v1      app-v1          1               2022-10-02 15:22:25.583786681 +0000 UTC deployed      my-chart-0.1.0   1.16.0     
v2      app-v1          1               2022-10-02 15:23:12.0213502 +0000 UTC   deployed      my-chart-0.1.0   1.16.0     
v3      app-v2          1               2022-10-02 15:23:31.282080767 +0000 UTC deployed      my-chart-0.1.0   1.16.0    
```
