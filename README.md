НАСТРОЙКА VLAN И TRUNK

1. НАСТРОЙКА TRUNK НА РОУТЕРЕ

Создать дополнительный сетевой адаптер на роутере (например ens19).

Физическому интерфейсу IP не назначать.

Файл:

/etc/net/ifaces/ens19/options

TYPE=eth
ONBOOT=yes
CONFIG_IPV4=no

Это trunk-интерфейс, через который будут проходить все VLAN.

---

2. СОЗДАНИЕ VLAN

Для каждого VLAN создать отдельную папку:

/etc/net/ifaces/ens19.100
/etc/net/ifaces/ens19.200

Пример VLAN100:

Файл:

/etc/net/ifaces/ens19.100/options

TYPE=vlan
HOST=ens19
VID=100
ONBOOT=yes
CONFIG_IPV4=yes

Файл:

/etc/net/ifaces/ens19.100/ipv4address

192.168.100.1/26

Пример VLAN200:

Файл:

/etc/net/ifaces/ens19.200/options

TYPE=vlan
HOST=ens19
VID=200
ONBOOT=yes
CONFIG_IPV4=yes

Файл:

/etc/net/ifaces/ens19.200/ipv4address

192.168.200.1/28

Перезапустить сеть:

systemctl restart network

Проверка:

ip a

Должны появиться:

ens19.100
ens19.200

---

3. НАСТРОЙКА PROXMOX

Создать отдельный bridge для внутренней сети.

Например:

vmbr2

К нему подключать:

* роутер;
* серверы;
* клиентов.

---

4. НАСТРОЙКА РОУТЕРА В PROXMOX

Для интерфейса, который смотрит во внутреннюю сеть:

Bridge: vmbr2
VLAN Tag: пусто

Почему?

Роутер должен видеть все VLAN одновременно.

Поэтому тег не назначается.

Это trunk.

---

5. НАСТРОЙКА СЕРВЕРА

Сетевой адаптер:

Bridge: vmbr2
VLAN Tag: номер VLAN

Например:

Bridge: vmbr2
VLAN Tag: 100

IP адрес выдаётся из сети VLAN100.

Шлюзом указывается IP интерфейса VLAN100 на роутере.

---

6. НАСТРОЙКА КЛИЕНТА

Сетевой адаптер:

Bridge: vmbr2
VLAN Tag: номер VLAN

Например:

Bridge: vmbr2
VLAN Tag: 200

IP адрес выдаётся из сети VLAN200.

Шлюзом указывается IP интерфейса VLAN200 на роутере.

---

7. ЛОГИКА

Роутер:
Bridge = vmbr2
VLAN Tag = пусто

Сервер:
Bridge = vmbr2
VLAN Tag = нужный VLAN

Клиент:
Bridge = vmbr2
VLAN Tag = нужный VLAN

На роутере создаются интерфейсы VLAN и назначаются IP-адреса шлюзов.

На клиентах и серверах VLAN-интерфейсы внутри Linux создавать не нужно — VLAN назначается через VLAN Tag в Proxmox.

---

8. ПРОВЕРКА

На роутере:

ip a

Проверить наличие интерфейсов:

ens19.100
ens19.200

На клиентах:

ping IP шлюза своего VLAN

Если шлюз пингуется — VLAN настроен корректно.
