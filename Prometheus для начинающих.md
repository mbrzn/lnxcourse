---
type: course
---
- [Пивотто. 2023. *Запускаем Prometheus*](zotero://select/items/_W8QD3ZIX)
## Часть I. ВВЕДЕНИЕ
### Глава 2. Настройка и запуск Prometheus 
#### Запуск Prometheus 
>По умолчанию Pro me theus использует TCP­порт 9090, а конфигурация выше предписывает извлекать данные каждые 10 с. Теперь вы можете запустить двоичный файл Pro me theus, выполнив команду ./prometheus:[^1]

```bash
sudo systemctl status prometheus
[sudo] password for berezin:
● prometheus.service - Prometheus Monitoring
     Loaded: loaded (/etc/systemd/system/prometheus.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2024-01-15 05:56:47 UTC; 11h ago
   Main PID: 729 (prometheus)
      Tasks: 10 (limit: 4558)
     Memory: 80.1M
...

/usr/local/bin/prometheus --help
usage: prometheus [<flags>]

The Prometheus monitoring server
...
```

#### Использование браузера выражений
> Теперь можно получить доступ к пользовательскому интерфейсу Prometheus в браузере по адресу http://localhost:9090/. Он выглядит, как показано на рис. 2.2. `http://192.168.1.100:9090/`
![[git/lnxcourse/files/Prometheus для начинающих.jpg]]

>Это браузер выражений (expression browser), в котором можно запускать запросы PromQL. В пользовательском интерфейсе также есть еще несколько страниц, которые помогут вам понять, что делает Pro me theus, например `страница Targets (Цели) на вкладке Status (Состояние)`, которая выглядит, как показано на рис. 2.3

![[git/lnxcourse/files/Prometheus для начинающих-1.jpg]]
	Здесь приложение `node_exporter` настроено с ошибками в конфигурационном файле `/etc/systemd/system/node_exporter.service`. [[git/lnxcourse/Мониторинг. Administrator Linux Basic. Otus#установить node_exporter]]

>Еще одна страница, на которую следует обратить внимание, – `страница  /metrics`. Тот факт, что сама система Pro me theus инструментирована и подготовлена для сбора метрик, не должен вызывать удивления. Метрики доступны по адресу http://localhost:9090/metrics в удобочитаемом виде, как показано на рис. 2.4.

![[git/lnxcourse/files/Prometheus для начинающих-2.jpg]]

>Для начала убедитесь, что вы находитесь в представлении консоли, введи-
те выражение up и щелкните по кнопке Execute (Выполнить).
Как показано на рис. 2.5, этот запрос возвращает единственный результат со 
значением 1 и именем up{instance="localhost:9090",job="prometheus"}. up – это 
специальная метрика, добавляемая Pro me theus перед началом сбора дан-
ных; 1 означает, что сбор прошел успешно. instance – это метка, идентифици-
рующая цель, откуда были извлечены данные. В данном случае она сообщает, 
что целью был сам сервер Pro me theus.[^2]

![[git/lnxcourse/files/Prometheus для начинающих-3.jpg]]

>Далее выполним запрос process_resident_memory_bytes, как показано на рис. 2.6.

![[git/lnxcourse/files/Prometheus для начинающих-9.jpg]]

- Наш сервер Prometheus использует 96559104 байт памяти
- а сервер nginx по адресу192.168.1.102 использует 20979712 байт
На вкладке `Graph`:
![[git/lnxcourse/files/Prometheus для начинающих-10.jpg]]

#### Запуск экспортера узла 
>Экспортер узла (Node Exporter) предоставляет метрики уровня ядра систем Unix, таких как Linux[^3].

>Вы можете загрузить готовую версию Node Exporter со страницы загрузки Pro me theus (https://oreil.ly/Bc4js). Перейдите на эту страницу и загрузите последнюю версию Node Exporter для ОС Linux с аппаратной архитектурой amd64.
>Архив необходимо распаковать, но файл конфигурации не требуется, поэтому экспортер узла можно запустить сразу же после распаковки:

>Теперь можно обратиться к Node Exporter, открыв браузере страницу `http://localhost:9100/ и перейдя к конечной точке /metrics`.
![[git/lnxcourse/files/Prometheus для начинающих-2.jpg]]

>Чтобы заставить Pro me theus выбирать метрики из Node Exporter, нужно добавить в prometheus.yml дополнительную конфигурацию выборки данных:[^4]
- [[git/lnxcourse/Мониторинг. Administrator Linux Basic. Otus#установить prometheus]]
```yaml
### /etc/prometheus/prometheus.yml
global:
  scrape_interval: 10s
scrape_configs:
 - job_name: prometheus
   static_configs:
    - targets:
       - localhost:9090
  - job_name: node ❶  static_configs:
    - targets:
       - localhost:9100
```
> Перезапустите Prometheus, чтобы ввести в действие новую конфигурацию. Для этого нажмите комбинацию Ctrl+C в консоли, где был запущен сервер Prometheus, а затем снова запустите его.

```bash
ls -l /usr/local/bin/node_exporter
-rwxr-xr-x 1 node_exporter node_exporter 20025119 янв 14 04:30 /usr/local/bin/node_exporter

sudo systemctl status node_exporter
● node_exporter.service - Node Exporter
     Loaded: loaded (/etc/systemd/system/node_exporter.service; disabled; vendor preset: enabled)
     Active: active (running) since Mon 2024-01-15 18:23:20 UTC; 10s ago
   Main PID: 44802 (node_exporter)
      Tasks: 5 (limit: 4558)
# служба активна

```
- [[git/lnxcourse/Мониторинг. Administrator Linux Basic. Otus#установить node_exporter]]
Имеются две цели в состоянии `up`:
![[git/lnxcourse/files/Prometheus для начинающих-4.jpg]]

#### Оповещение 

## Часть II. МОНИТОРИНГ ПРИЛОЖЕНИЙ 

### Глава 6. Создание дашбордов с Grafana 
#### Установка сервера Grafana
```bash
sudo docker pull grafana/grafana:9.1.6
...
Status: Downloaded newer image for grafana/grafana:9.1.6
docker.io/grafana/grafana:9.1.6

curl http://localhost:3000/
<a href="/login">Found</a>.
```
#### Источники данных 
#### Дашборды и панели 
#### Как избежать «стены графиков» 
#### Панель временных рядов 
#### Элементы управления временем 
#### Панель статистики 
#### Панель таблицы 
#### Панель временнуй шкалы состояний 
#### Переменные шаблона 

## Часть III. МОНИТОРИНГ ИНФРАСТРУКТУРЫ 
### Глава 7. Node Exporter 
- `http://192.168.1.100:9100/`
![[git/lnxcourse/files/Prometheus для начинающих-17 1.jpg]]
#### Сборщик информации о процессоре 
#### Сборщик информации о файловой системе 
#### Сборщик дисковой статистики 
#### Сборщик информации о сетевых устройствах 
#### Сборщик информации о потреблении памяти 
#### Сборщик информации об аппаратной части 
#### Сборщик статистики
#### Сборщик информации об имени узла 
#### Сборщик информации об операционной системе 
#### Сборщик информации о средней нагрузке 
#### Сборщик информации о давлении 
#### Сборщик текстовых файлов 
#### Использование сборщика текстовых файлов 
#### Отметки времени 

### Глава 8. Обнаружение служб 
#### Механизмы обнаружения служб 
##### Статическое обнаружение служб 
>Вы уже видели статическую конфигурацию в главе 2, где цели задаются непосредственно в prometheus.yml. Такую конфигурацию можно использовать, если окружение простое и редко меняется. Это может быть ваша домашняя сеть, окружение с локальным Pushgateway или даже окружение с Prometheus, как в примере 8.1.[^5]

...
- пример локальной сети [[мониторинг nginx#настроить `prometheus` сервер]]

---

>Пример 8.3    Здесь определяются две цели для мониторинга, каждая в своей 
статической конфигурации
```yaml

scrape_configs:
 - job_name: node
   static_configs:
     - targets: ❶      
       - host1:9100
      labels:
        datacenter: paris
     - targets: ❷      
       - host2:9100
       - host3:9100
      labels:
        datacenter: montreal
```
❶ вП емревтакяе  сdтaаtaтcиeчnеtсeкr.ая конфигурация с единственной целью и со значением paris 
❷ dВaтtоaрcаeяn tсeтrа.ти


##### Обнаружение служб на основе файла 
##### Обнаружение служб через HTTP 
##### Обнаружение служб с помощью Consul 
##### Обнаружение служб с помощью ЕС2 
#### Изменение меток 
##### Выбор целей для мониторинга 
##### Метки целей 
#### Как извлекать метрики 
##### metric_relabel_configs
##### Конфликты меток и honor_labels

### Глава 9. Контейнеры и Kubernetes 
#### cAdvisor 
>Все компоненты Prometheus успешно работают в контейнерах. Единственное исключение – `Node Exporter`[^6]

>`cAdvisor` во многом похож на `Node Exporter`. Последний предоставляет метрики о машине, а первый – экспортирует метрики о cgroups. `Cgroups` – это механизм изоляции ядра Linux, который обычно используется для реализации контейнеров в Linux, а также такими средами выполнения, как systemd.
>Вы можете запустить `cAdvisor` с помощью Docker:
```bash
docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:rw \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --volume=/dev/disk/:/dev/disk:ro \
  --publish=8081:8080 \
  --detach=true \
  --name=cadvisor \
  gcr.io/cadvisor/cadvisor:v0.45.0
```
>Метрики контейнеров имеют префикс container_, и, как нетрудно заметить, все они имеют метку id. Метки id, начинающиеся с /docker/ или /system.slice/docker-, взяты из Docker и его контейнеров

`http://192.168.1.102:8081/docker/`
![[git/lnxcourse/files/Prometheus для начинающих-18.jpg]]

>а метки id со значениями /user.slice/ и /system.slice/, берутся из systemd. Если у вас есть другое программное обеспечение, использующее контрольные группы (cgroups), то его контрольные группы тоже будут присутствовать в списке.

`http://192.168.1.102:8081/containers/`:
![[git/lnxcourse/files/Prometheus для начинающих-19.jpg]]
![[git/lnxcourse/files/Prometheus для начинающих-19.jpg]]

>Эти метрики можно получить с помощью prometheus.yml, например:
```yaml
scrape_configs:
  - job_name: cadvisor
    scrape_interval: 5s
    static_configs:
      - targets:
          - 192.168.1.102:8081
```

##### Метрики потребления процессора 
- метрика `container_cpu_usage_seconds_total{job="cadvisor"}`
![[git/lnxcourse/files/Prometheus для начинающих-20.jpg]]
##### Метрики потребления памяти 
>На практике` container_memory_working_set_bytes` наиболее близок к RSS. Также можно следить за container_memory_usage_bytes, так как эта метрика включает кеш страниц. В общем случае мы рекомендуем полагаться на такие метрики, как process_resident_memory_bytes, из самого процесса, а не на метрики из контрольных групп cgroups.[^7] 

>Если ваши приложения не предоставляют метрик для Pro me­theus, то cAdvisor может оказаться хорошей временной мерой, чьи метрики с успехом можно использовать для нужд отладки и профилирования.

- ` container_memory_working_set_bytes {job="cadvisor"}` ![[git/lnxcourse/files/Prometheus для начинающих-21.jpg]]

##### Метки 
#### Kubernetes 
##### Запуск в Kubernetes 
##### Обнаружение служб 
##### kube­state­metrics 
#### Альтернативные развертывания 

## Часть V. ОПОВЕЩЕНИЕ
### Глава 18. Оповещение
#### Правила оповещения
##### for
##### Метки оповещений
##### Аннотации и шаблоны
##### Что такое хорошее оповещение?
#### Настройка диспетчеров уведомлений в Prometheus
##### Внешние метки

### Глава 19. Alertmanager 
#### Конвейер уведомлений 
#### Конфигурационный файл 
##### Дерево маршрутизации
##### Приемники
##### Подавление
#### Веб­интерфейс Alertmanager

[^1]: [Пивотто. 2023. *Запускаем Prometheus*](zotero://select/items/_W8QD3ZIX) с. 36
[^2]: [Там же с. 39](zotero://select/items/_W8QD3ZIX) с. 36
[^3]: [Там же с. 42](zotero://select/items/_W8QD3ZIX) с. 36
[^4]: [Там же с. 43](zotero://select/items/_W8QD3ZIX) с. 36
[^5]: [Там же с. 147](zotero://select/items/_W8QD3ZIX) с. 36
[^6]: [Там же с. 170](zotero://select/items/_W8QD3ZIX) с. 36
[^7]: [Там же с. 172](zotero://select/items/_W8QD3ZIX) с. 36