# lighthouse-role

Ansible-роль устанавливает [LightHouse](https://github.com/VKCOM/lighthouse) и настраивает nginx для раздачи его веб-интерфейса.

## Что делает роль

- Устанавливает nginx и утилиты для управления контекстами SELinux через `dnf`.
- Загружает архив LightHouse, распаковывает его и копирует файлы в каталог установки.
- Назначает файлам контекст SELinux `httpd_sys_content_t` и применяет его.
- Создаёт конфигурацию nginx: слушает порт 80, раздаёт файлы из каталога LightHouse и направляет неизвестные пути на `index.html` (поддержка клиентской маршрутизации).
- Запускает nginx и включает его автозапуск. При изменении файлов приложения или конфигурации nginx перезапускает службу.

## Требования

- Целевая система с `dnf` и `systemd` (например, семейство RHEL/Fedora).
- Ansible 2.20.1 или новее.
- Коллекция `community.general` для модуля `sefcontext`.
- Доступ целевого хоста к GitHub для загрузки архива.

## Использование

Для использования необходимо роль в playbook:

```yaml
---
- name: Установить LightHouse
  hosts: lighthouse
  become: true
  roles:
    - lighthouse-role
```

Затем указать целевые хосты в inventory и запустить playbook, например:

```sh
ansible-playbook -i inventory.ini site.yml
```

## Переменные роли

| Переменная | Значение по умолчанию | Назначение |
| --- | --- | --- |
| `lighthouse_version` | `master` | Заявленная версия LightHouse. В текущей реализации загрузка использует URL `lighthouse_repo`, поэтому эта переменная сама по себе на версию не влияет. |
| `lighthouse_install_dir` | `/var/www/lighthouse` | Каталог, из которого nginx раздаёт сайт. |
| `lighthouse_archive` | `/tmp/lighthouse.tar.gz` | Путь для загружаемого архива. |
| `lighthouse_repo` | `https://github.com/VKCOM/lighthouse/archive/refs/heads/master.tar.gz` | URL архива LightHouse. Чтобы выбрать другую ветку или архив, переопределите URL. |


## Настройка nginx

Роль устанавливает `/etc/nginx/conf.d/lighthouse.conf`: nginx слушает порт 80 как сервер по умолчанию и использует `lighthouse_install_dir` в качестве корня сайта. Необходимо убедиться, что порт 80 доступен клиентам и не занят другой конфигурацией nginx.
