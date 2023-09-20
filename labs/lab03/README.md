#  DHCP

###  Задание:
 1. Создание сети и настройка основных параметров устройств.
 2. Конфигурирование и проверка двух DHCPv4 серверов на R1.
 3. Конфигурирование и проверка DHCPv4 сервера на R2.


### 1 Создание сети и настройка основных параметров устройства.

#### 1.1 Таблица адресации.

 | Device        | Interface     | IP address    | Subnet mask     | Default gateway |
 | ------------- | ------------- | ------------- | --------------- | --------------- |
 | R1            | e0/0          | 10.0.0.1      | 255.255.255.252 | N/A             |
 |               | e0/1          | N/A           | N/A             |                 |
 |               | e0/1.100      |               |                 |                 |
 |               | e0/1.200      |               |                 |                 |
 |               | e0/1.1000     |               |                 |                 |
 | S1            | VLAN 200      |               |                 |                 |
 | S2            | VLAN 1        |               |                 |                 |
 | PC-A          | e0            | DHCP          | DHCP            | DHCP            |
 | PC-B          | e0            | DHCP          | DHCP            | DHCP            |


#### 1.2 Таблица VLAN.

  | VLAN         | Name        | Interface assigned             |
  | ------------ | ----------- | ------------------------------ |
  | 1            | N/A         | S2: e0/1                       |
  | 100          | Clients     | S1: e0/1                       |
  | 200          | Management  | S1: VLAN200                    |
  | 999          | X           | S1: e0/2-3                     |
  | 1000         | Native      | N/A                            |

#### 1.3 Создание сети.

  В eve-ng создана сеть следующей топологии:
  ![](img/lab03.png)

#### 1.4 Настройка основных параметров устройств.
##### Настройка S1, S2, R1 и R2 на примере S1.

 - a. Присвойте имена устройствам в соответствии с топологией.
 ```
 Switch(config)#hostname S1
 ```
 - b. Отключите поиск DNS.
 ```
 Switch(config)#no ip domain-lookup
 ```
 - c. Назначьте class в качестве зашифрованного пароля доступа к привилегированному режиму.
 ```
 S1(config)#service password-encryption
 S1(config)#enable secret class
 ```
 - d. Назначьте cisco в качестве паролей консоли и VTY и активируйте вход для консоли и VTY каналов.
 ```
 S1(config)#line console 0
 S1(config-line)#password cisco
 S1(config-line)#login
 S1(config)#exit
 S1(config)#line vty 0 4
 S1(config-line)#password cisco
 S1(config-line)#login
 S1(config)#exit
 ```
 - e. Настройте logging synchronous для консольного канала.
 ```
 S1(config)#line console 0
 S1(config-line)#logging synchronous
 ```
 - f. Настройте баннерное сообщение дня (MOTD) для предупреждения пользователей о запрете несанкционированного доступа.
 ```
 S1(config)#banner motd "This is a secure system. Authorized Access Only!"
 ```

 - g. Скопируйте текущую конфигурацию в файл загрузочной конфигурации.
 ```
 S1#copy running-config startup-config
 Destination filename [startup-config]?
 Building configuration...
 Compressed configuration from 983 bytes to 719 bytes[OK]
 ```

 - h. Установите текущую дату и время.
 ```
 S1#clock set 19:45:55 sep 20 2023
 ```

#### 1.3 Проверка связи коммуторов.

 - a. Проверка пингом с S1 на S2:
 ```
  S1#ping 192.168.1.2
  Type escape sequence to abort.
  Sending 5, 100-byte ICMP Echos to 192.168.1.2, timeout is 2 seconds:
  !!!!!
  Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
  ```
 - b. Проверка пингом с S1 на S3:
 ```
  S1#ping 192.168.1.3
  Type escape sequence to abort.
  Sending 5, 100-byte ICMP Echos to 192.168.1.3, timeout is 2 seconds:
  !!!!!
  Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/5 ms
 ```
 - c. Проверка пингом с S2 на S3:
 ```
  S2#ping 192.168.1.3
  Type escape sequence to abort.
  Sending 5, 100-byte ICMP Echos to 192.168.1.3, timeout is 2 seconds:
  !!!!!
  Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
 ```

### 2. Определение корневого моста
 - a. Отключить все порты на коммутаторах
```
S1(config)#interface range Et0/0-3    
S1(config-if-range)#shutdown
```
 - b. Настройте подключенные порты в качестве транковых.
 ```
 S1(config-if-range)#switchport trunk encapsulation dot1q
 S1(config-if-range)#switchport mode trunk
 ```
 - c. Включите порты e0/1 и e0/3 на всех коммутаторах.
 ```
 S1(config)#interface range ethernet 0/1, ethernet 0/3
 S1(config-if-range)#no shutdown
 ```
 - d. Отобразите данные протокола spanning-tree.

 ```
 S1#show spanning-tree

   Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
              Address     aabb.cc00.1000

 S2#show spanning-tree

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.2000

 S3#show spanning-tree

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.3000

 ```

 - e. Схема с ролями портов.
 ![](img/lab02_1.png)

 - f. Вопросы по схеме.
    - Какой коммутатор является корневым мостом: S1.
    - Почему этот коммутатор был выбран протоколом spanning-tree в качестве корневого моста:
    При определении корневого коммутатора вычисляется Bridge ID, состоящий из: приоритета коммутатора, номера VLAN и значения MAC-адреса устройства. Первые два параметра у коммутаторов одинаковые, но у S1 наименьший MAC. В связи с чем он и был выбран.
    - Какие порты на коммутаторе являются корневыми портами: S2/e0/1 и S3/e0/3.
    - Какие порты на коммутаторе являются назначенными портами: S1/e0/1, S1/e0/3, S2/e0/3.
    - Какой порт отображается в качестве альтернативного и в настоящее время заблокирован: S3/e0/1
    - Почему протокол spanning-tree выбрал этот порт в качестве невыделенного (заблокированного) порта?
    Когда S2 и S3 решали, который из портов, ноторыми они соединяются, будет Desig, а кто Altn, был проверен Root Path Cost обоих коммутаторов. Он оказался равен. Тогда выбор сделать порт назначенным пал на S2 из-за меньшего BID коммутатора, в связи с меньшим значением MAC-адреса. Оставшийся порт на S3 был заблокирован.

### 3. Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов.
 - Определите коммутатор с заблокированным портом.
 Заблокирован порт S3/e0/1.
 ```
Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Altn BLK 100       128.2    Shr
Et0/3               Root FWD 100       128.4    Shr
 ```
 - Измените стоимость порта.
```
S3(config)#interface e0/3
S3(config-if)#spanning-tree cost 18
```
 - Просмотрите изменения протокола spanning-tree.
```
Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Desg LIS 100       128.2    Shr
Et0/3               Root FWD 18        128.4    Shr
```
- Почему протокол spanning-tree заменяет ранее заблокированный порт на назначенный порт и блокирует порт, который был назначенным портом на другом коммутаторе?
  Изменилась Root Path Cost для S3, который теперь стал лучше по первому критерию и сделал свой порт назначенным.

 - Удалите изменения стоимости порта.
 ```
 S3(config)#interface e0/3
 S3(config-if)#no spanning-tree cost 18

 Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Altn BLK 100       128.2    Shr
Et0/3               Root FWD 100       128.4    Shr
 ```
### 4. Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов.
 - Включение портов e0/0 и e0/2 на всех коммутаторах.
 ```
 S1(config)#interface range ethernet 0/0, ethernet 0/2
 S1(config-if-range)#no shutdown
 ```
 - Какой порт выбран протоколом STP в качестве порта корневого моста на каждом коммутаторе некорневого моста?
  Корневыми выбраны порты: S2/e0/0 и S3/e0/2.
 - Почему протокол STP выбрал эти порты в качестве портов корневого моста на этих коммутаторах?
  При равенстве Root Path cost и BID, как в данном случае, выбирается порт на коммутаторе в сторону рута, который имеет наименьший номер (или Prio.Nbr).

### Вопросы для повторения.
 - 1. Какое значение протокол STP использует первым после выбора корневого моста, чтобы определить выбор корневого порта?
  Root Path Cost.
 - 2. Если первое значение на двух портах одинаково, какое следующее значение будет использовать протокол STP при выборе порта?
  BID соседнего коммутатора за этим портом.
 - 3. Если оба значения на двух портах равны, каким будет следующее значение, которое использует протокол STP при выборе порта?
  Номер порта на соседним коммутаторе за этим портом.
