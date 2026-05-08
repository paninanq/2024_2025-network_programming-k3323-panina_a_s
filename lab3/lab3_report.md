University: [ITMO University](https://itmo.ru/ru/)
Faculty: [FICT](https://fict.itmo.ru)
Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)
Year: 2025/2026
Group: K3323
Author: Panina Anna Sergeevna
Lab: Lab1
Date of create: 07.05.2026
Date of finished: 08.05.2026

# Лабораторная работа №3

## Задание

<https://itmo-ict-faculty.github.io/network-programming/education/labs2023_2024/lab3/lab3/>

### Netbox

netbox я подняла на вм с помошью docker compose:

```
sudo apt update
sudo apt install -y git curl ca-certificates docker.io docker-compose-v2

sudo systemctl enable --now docker
sudo usermod -aG docker $USER
cd ~
git clone -b release https://github.com/netbox-community/netbox-docker.git
cd netbox-docker

cp docker-compose.override.yml.example docker-compose.override.yml

docker compose pull
docker compose up -d
```

Создала админа:
```
docker compose exec netbox /opt/netbox/netbox/manage.py createsuperuser
```
Добавила два моих chr:

![netbox](images/pic1.png)
![netbox](images/pic2.png)
![netbox](images/pic3.png)

### Ansible 

Первым делом я настроила динамический inventory ["dynamic inventory"](./inventories/netbox/netbox_inventory.yml)

![Файлы](images/pic4.png)

#### Сценарий забора данных из netbox

["export_netbox"](.playbooks/export_netbox.yml)

Данный playbook обращается к API NetBox. Он забирает основные объекты, которые нужны для дальнейшей работы: устройства, интерфейсы, IP-адреса, префиксы, роли, платформы и теги. Дополнительно используется lookup из коллекции `netbox.netbox`.

После выполнения плейбука информация собрана в файл lab3/out/netbox_dump.json.

["output_netbox"](.out/netbox_dump.json)

#### Cценарий, при котором на основе данных из Netbox можно настроить 2 CHR, изменить имя устройства, добавить IP адрес на устройство

["configure_chr_from_netbox"](.playbooks/configure_chr_from_netbox.yml)

Этот playbook берет данные об устройствах из NetBox и применяет их к CHR через RouterOS CLI. Сначала он находит устройство в NetBox по имени из inventory, затем получает назначенные IP-адреса и формирует желаемую конфигурацию. В безопасном режиме playbook только показывает, какие изменения будут выполнены, а для реального применения используется переменная `apply_changes=true`.

Я поменяла имя роутера 

![chr](images/pic5.png)

И прокатила мой плейбук, после чего как и ожидалось имя изменилось
![chr](images/pic6.png)

#### Cценарий, позволяющий собрать серийный номер устройства и вносящий серийный номер в Netbox

["sync_serial_to_netbox"](.playbooks/sync_serial_to_netbox.yml)

Этот playbook подключается к CHR, получает идентификатор устройства и при необходимости записывает его обратно в NetBox. Для CHR серийный номер берется из вывода команды `/system license print`. По умолчанию playbook работает в режиме проверки, а обновление NetBox выполняется при запуске с переменной `update_netbox=true`.

После проката плейбука получаю запись серийного номера в netbox

![serial](images/pic7.png)
![serial](images/pic8.png)


### Вывод

В ходе работы был развернут netbox и написано три сценария ansible, которые могут забрать информацию об устройстве из netbox, поправить устройство и сделать записи в netbox.
