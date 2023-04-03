# Ansible-роль `docker`

Роль для установки [Docker](https://docs.docker.com/engine/install/).

[Docker Compose](https://docs.docker.com/compose/) устанавливается как плагин к Docker (`docker compose version`).

## Аргументы роли

### Обязательные

| Аргумент | Описание
| --- | ---

### Необязательные

<table>
<thead>
<th>
Аргумент
</th>
<th>
Описание
</th>
<th>
Default
</th>
</thead>
<tbody>

<tr>

<td valign="top">

`daemon_json_extra`

</td>
<td valign="top">

Параметры [dockerd](https://docs.docker.com/config/daemon/), которые будут перекрывать или дополнять следующие
параметры по-умолчанию (хранятся в приватной переменной роли [`_daemon_json_default`](vars/main.yml)):

```yaml
builder:
  gc:
    defaultKeepStorage: '10GB'
    enabled: true
features:
  buildkit: true
log-driver: local
log-opts:
  max-size: 10m
  max-file: '3'  # must always be a string
```

Итоговые настройки dockerd хранятся в файле `/etc/docker/daemon.json`.
</td>

<td valign="top">

`{}`

</td>

<tr>

<td valign="top">

`remove_unused_images`

</td>
<td valign="top">

Удалять ли неиспользуемые образы при очистке Docker'а (см. ниже)

</td>

<td valign="top">

`true`

</td>

</tr>

</tbody>
</table>

## Автоматическая очистка Docker

Роль настраивает _ежедневную_ очистку неиспользуемых данных Docker:

- остановленные **контейнеры**
- неиспользуемые **сети**
- зависшие (dangling) и неиспользуемые **образы**
- неиспользуемые **тома** (volumes)

Локальный **кэш сборки** очищается самим BuildKit-демоном через механизм
[Garbage collection](https://docs.docker.com/build/cache/garbage-collection/).

Для предотвращения сюрпризов, удаляются только те неиспользуемые контейнеры, сети
и образы, с момента _создания_ которых прошло не менее **24 часов**. Для образов
учитывается именно время создания, а не закачки (pull).

Неиспользуемые тома удаляются без какого-либо ограничения по времени. Во-первых,
в Docker не реализована такая возможность. Во-вторых, предполагается, что вся
чувствительная информация не должна храниться с помощью механизма томов.

## Пример

```yaml
- role: docker
  vars:
    daemon_json_extra:
      log-driver: json-file
      log-opts:
        max-size: 100m
        max-file: '5'  # must always be a string
        tag: '{% raw %}{{.ImageName}}|{{.Name}}{% endraw %}'
      # Для сбора метрик Prometheus'ом
      experimental: true
      metrics-addr: "0.0.0.0:9323"
      # Для устранения конфликтов с VPN-сетями
      bip: 100.64.0.1/24
      fixed-cidr: 100.64.0.1/24
      default-address-pools:
        - base: 100.65.0.0/16
          size: 25
      # Docker Registry в офисной VPN-сети
      insecure-registries:
        - 172.17.21.6:4444
        - 172.17.21.6:2222
      registry-mirrors:
        - http://172.17.21.6:4444
```

## Ссылки

- [dockerd options](https://docs.docker.com/engine/reference/commandline/dockerd/)
- [Garbage collection](https://docs.docker.com/build/cache/garbage-collection/)
- [Configure logging drivers](https://docs.docker.com/config/containers/logging/configure/)
- [Customize log driver output](https://docs.docker.com/config/containers/logging/log_tags/) (`log-opts.tag` options)
