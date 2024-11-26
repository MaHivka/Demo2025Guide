

## Задание 4. Настройте на интерфейсе HQ-RTR в сторону офиса HQ виртуальный коммутатор.
- Сервер HQ-SRV должен находиться в ID VLAN 100;
- Клиент HQ-CLI в ID VLAN 200;
- Создайте подсеть управления с ID VLAN 999;
- Основные сведения о настройке коммутатора и выбора реализации разделения на VLAN занесите в отчёт.

1. Настройка VLAN:
    - **1 способ**. nmtui (установка на альт: apt install NetworkManager-tui):
    - Добавляем интерфейс ![Добавление интерфейса](https://github.com/MaHivka/Demo2025Guide/blob/main/Модуль%201/VLAN/VLAN_nmtui_1.png)
    - Настраиваем интерфейс ![Настройка интерфейса](https://github.com/MaHivka/Demo2025Guide/blob/main/Модуль%201/VLAN/VLAN_nmtui_2.png)
    - **2 способ**. nmcli (статика):
    ```
    nmcli con add type vlan con-name <Название подключения> ifname <Название интерфейса> id <Номер VLAN> dev <Имя физ интерфейса, пример: ens3> ip4 <ip адрес инетрфейса/маска> gw4 <ip адрес шлюза>
    ```
    - **3 способ**. NetworkManager GUI:
    - Добавляем интерфейс ![Добавление интерфейса](https://github.com/MaHivka/Demo2025Guide/blob/main/Модуль%201/VLAN/VLAN_nmGUI_1.png)
    - Настраиваем номер VLAN и родительский интерфейс ![Настройка VLAN](https://github.com/MaHivka/Demo2025Guide/blob/main/Модуль%201/VLAN/VLAN_nmGUI_2.png)
    - Настройка ip адреса (Можно оставить DHCP или задать статику) ![Настрйка IP](https://github.com/MaHivka/Demo2025Guide/blob/main/Модуль%201/VLAN/VLAN_nmGUI_3.png)


## Задание 8. Настройка динамической трансляции адресов.
- Настройте динамическую трансляцию адресов для обоих офисов;
- Все устройства в офисах должны иметь доступ к сети Интернет.

### Решение:
1. Настройка NAT на ISP (Alt):
   - Установка iptables:
   ```
   apt-get install iptables
   ```
   - Настройка iptables:
   Шаблон:
   ```
   iptables -t nat -A POSTROUTING -o <Интерфейс в интернет> -s <Внутренняя подсеть/маска> -j MASQUERADE
   ```
   Пример для двух подсетей:
   ```
   iptables -t nat -A POSTROUTING -o ens3 -s 172.16.4.0/28 -j MASQUERADE
   iptables -t nat -A POSTROUTING -o ens3 -s 172.16.5.0/28 -j MASQUERADE
   ```
   Включаем сервис iptables и ставим его в автозагрузку:
   ```
   systemctl enable --now iptables
   ```
   После настройки NAT и включения iptables на ISP EcoRouter-ы смогут пинговать "Интернет" и друг друга.
   Чтобы после перезапуска ВМ iptables сохранил NAT, нужно прописать следующую команду:
   ```
   iptables-save -f /etc/sysconfig/iptables
   ```
2. Настройка NAT на EcoRouter:
   - Настройка NAT на интерфейсах:
   ```
    int <Внешний интерфейс>
        ip nat outside
    int <Внутренний интерфейс>
        ip nat inside
   ```
   - Создание пула адресов для NAT:
   ```
    ip nat pool <Название пула> <Начальный адрес сети>-<Конечный адрес сети>,<Начальный адрес сети>-<Конечный адрес сети>, ...
   ```
   - Пример для HQ-RTR:
   ```
    int eth0
        ip nat outside
    int int1
        ip nat inside
    int int2
        ip nat inside
    
    ip nat pool nat_pool 192.168.100.1-192.168.100.62,192.168.200.1-192.168.200.14
   ```
