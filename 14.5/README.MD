# Домашнее задание к занятию "14.5 SecurityContext, NetworkPolicies"

## Задача 1: Рассмотрите пример 14.5/example-security-context.yml

Создайте модуль

```
kubectl apply -f 14.5/example-security-context.yml
```

Проверьте установленные настройки внутри контейнера

```
kubectl logs security-context-demo
uid=1000 gid=3000 groups=3000
```

<details><summary>Запуск пода из манифеста:</summary>

```

@p:~/14.5$ kubectl apply -f example-security-context.yml
pod/security-context-demo created
p@p:~/14.5$ kubectl get pods
NAME                    READY   STATUS      RESTARTS        AGE
curl                    1/1     Running     1 (2m38s ago)   10d
fedora                  0/1     Completed   0               10d
security-context-demo   1/1     Running     0               10s
p@p:~/14.5$ kubectl get pods -o wide
NAME                    READY   STATUS      RESTARTS        AGE   IP            NODE       NOMINATED NODE   READINESS GATES
curl                    1/1     Running     1 (3m58s ago)   10d   172.17.0.2    minikube   <none>           <none>
fedora                  0/1     Completed   0               10d   172.17.0.11   minikube   <none>           <none>
security-context-demo   1/1     Running     0               90s   172.17.0.10   minikube   <none>           <none>



```

</details>

<details><summary>Вход в контейнер и проверка:</summary>

```
p@p:~/14.5$ kubectl exec -it security-context-demo -- bash
bash-5.1$ id
uid=1000 gid=3000 groups=3000
bash-5.1$ whoami
whoami: cannot find name for user ID 1000



```

</details>


## Задача 2 (*): Рассмотрите пример 14.5/example-network-policy.yml

Создайте два модуля. Для первого модуля разрешите доступ к внешнему миру
и ко второму контейнеру. Для второго модуля разрешите связь только с
первым контейнером. Проверьте корректность настроек.


<details><summary>Создаю поды и сервисы</summary>

```

p@p:~/14.5/2$ for file in *pod*; do kubectl apply -f $file; done
pod/module1 created
pod/module2 created
p@p:~/14.5/2$ for file in *svc*; do kubectl apply -f $file; done
service/module1 created
service/module2 created
p@p:~/14.5/2$ kubectl get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP    12d
module1      ClusterIP   10.106.185.47   <none>        5978/TCP   16s
module2      ClusterIP   10.97.174.216   <none>        6379/TCP   15s
@p:~/14.5/2$ kubectl get pods
NAME                    READY   STATUS      RESTARTS      AGE
curl                    1/1     Running     1 (22m ago)   11d
fedora                  0/1     Completed   0             11d
module1                 1/1     Running     0             4m29s
module2                 1/1     Running     0             4m29s
security-context-demo   1/1     Running     0             20m


```

</details>


<details><summary>Проверяю, что у обоих модулей есть доступ друг к другу и в интернет. Убеждаюсь, что сетевые политики не настроены</summary>

```

p@p:~/14.5/2$ kubectl get networkpolicies.networking.k8s.io -A
No resources found
p@p:~/14.5/2$ kubectl exec -it module1 -- nc -vz -w 2 module2 6379
Connection to module2 6379 port [tcp/redis] succeeded!
p@p:~/14.5/2$ kubectl exec -it module1 -- nc -vz -w 2 yandex.ru 443
Connection to yandex.ru 443 port [tcp/https] succeeded!
p@p:~/14.5/2$ kubectl exec -it module2 -- nc -vz -w 2 module1 5978
module1 (10.106.185.47:5978) open
p@p:~/14.5/2$ kubectl exec -it module2 -- nc -vz -w 2 yandex.ru 443
yandex.ru (77.88.55.88:443) open


```

</details>


<details><summary>Применяю сетевые политики</summary>

```

p@p:~/14.5/2$ for file in *np*; do kubectl apply -f $file; done
networkpolicy.networking.k8s.io/default-deny-all created
networkpolicy.networking.k8s.io/netology-14-5-module1 created
networkpolicy.networking.k8s.io/netology-14-5-module2 created
p@p:~/14.5/2$ ls
00-pod-module1.yml  10-svc-module1.yml  20-np-default.yml  22-np-module2.yml
00-pod-module2.yml  10-svc-module2.yml  22-np-module1.yml
p@p:~/14.5/2$ nano 21-np-dns.yml
p@p:~/14.5/2$ kubectl apply -f 21-np-dns.yml 
networkpolicy.networking.k8s.io/allow-dns-access created


```

</details>


<details><summary>Проверяю, что сетевые политики настроены, потом проверяю результат: у первого пода есть доступ ко второму и в интернет, а у второго только к первому</summary>

```

@p:~/14.5/2$ kubectl get networkpolicies.networking.k8s.io -A
NAMESPACE   NAME                    POD-SELECTOR   AGE
default     allow-dns-access        <none>         41s
default     default-deny-all        <none>         2m32s
default     netology-14-5-module1   role=module1   2m32s
default     netology-14-5-module2   role=module2   2m32s
p@p:~/14.5/2$ kubectl exec -it module1 -- nc -vz -w 2 module2 6379
Connection to module2 6379 port [tcp/redis] succeeded!
p@p:~/14.5/2$ kubectl exec -it module1 -- nc -vz -w 2 yandex.ru 443
Connection to yandex.ru 443 port [tcp/https] succeeded!
p@p:~/14.5/2$ kubectl exec -it module2 -- nc -vz -w 2 module1 5978
Connection to module2 5978 port [tcp/*] succeeded!
p@p:~/14.5/2$ kubectl exec -it module2 -- nc -vz -w 2 google.com 443
nc: google.com (64.233.162.102:443): Operation timed out
command terminated with exit code 1

p@p:~/14.5/2$ cat  22-np-module2.yml 
cat: nano: Нет такого файла или каталога
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: netology-14-5-module2
spec:
  podSelector:
    matchLabels:
      role: module2
  policyTypes:
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: module2
  egress:
  - to:
    - podSelector:
        matchLabels:
          role: module1




```



---

### Как оформить ДЗ?

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

В качестве решения прикрепите к ДЗ конфиг файлы для деплоя. Прикрепите скриншоты вывода команды kubectl со списком запущенных объектов каждого типа (pods, deployments, statefulset, service) или скриншот из самого Kubernetes, что сервисы подняты и работают, а также вывод из CLI.

---
