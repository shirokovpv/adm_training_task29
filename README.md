# adm_training_task29
<h1 align="center">Занятие 29. Репликация postgres</h1>
<h3 class="western"><span style="font-family: Roboto, serif;"><span style="font-size: small;">Описание задания</span></span></h3>
<p><span style="font-weight: 300;">1) Настроить hot_standby репликацию с использованием слотов</span></p>
<p><span style="font-weight: 300;">2) Настроить правильное резервное копирование</span></p>
<h3 class="western"><a name="_heading=h.df570rpzx1qg"></a><span style="font-family: Roboto, serif;"><span style="font-size: small;">Используемые ОС</span></span></h3>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">Хостовая ОС Ubuntu 24.04 Desktop с установленным Vagrant 2.3.5. VirtualBox версия 7.0.26 r168464</span></span></p>
<h3 class="western"><span style="font-family: Roboto, serif;"><span style="font-size: small;">Выполнение</span></span></h3>
<p>******************************</p>
<p>Ввиду невозможности в нынешних реалиях воспользоваться стандартными репозиториями Vagrant (в РФ они сейчас не доступны), предложено обходное решение. Использован репозиторий, развернутый на&nbsp;<a href="https://vagrant.elab.pro/" rel="nofollow">https://vagrant.elab.pro/</a></p>
<p>Так как официальный пакет последней версии Vagrant также не доступен для скачивания, пакет взят оттуда же. Версия 2.3.5.</p>
<img width="411" height="88" alt="image" src="https://github.com/user-attachments/assets/7591c684-f543-4bf6-bfa8-3706a7648c97" />
<p>&nbsp;</p>
<p>Для подключения репозитория надо добавить в Vagrant-файл строку:</p>
<p><code>ENV['VAGRANT_SERVER_URL'] = 'https://vagrant.elab.pro'</code></p>
<p>И изменить box_name и box_version (как в репозитории, если туда зайти).</p>
<img width="238" height="274" alt="image" src="https://github.com/user-attachments/assets/aa6407df-e555-4e28-ac4f-7e8bb869c75c" />
<p>******************************</p>
<p><span style="font-weight: 300;">Создадим каталог ~/task29, а в нем Vagrantfile, в котором будут указаны параметры наших ВМ. Файл прикладываю сюда.</span></p>
<img width="450" height="862" alt="image" src="https://github.com/user-attachments/assets/191eafbf-76e4-4ff4-b941-d4b28c6062c0" />
<p>&nbsp;</p>
<p><span style="font-weight: 300;">Для удобства на все хосты можно установить утилиту telnet:&nbsp;</span></p>
<p><span style="font-weight: 300;"><code>apt install -y telnet</code></span></p>
<p><span style="font-weight: 300;">Команды должны выполняться от root-пользователя</span></p>
<p><span style="font-weight: 300;">Для перехода в root-пользователя вводим </span><span style="font-weight: 300;"><code>sudo -i</code></span></p>
<p><span style="font-weight: 300;">Перед настройкой репликации необходимо установить postgres-server на хосты node1 и node2:</span></p>
<p><span style="font-weight: 300;">1) Устанавливаем postgresql-server 14: </span><span style="font-weight: 300;"><code>apt install postgresql postgresql-contrib</code></span></p>
<p><span style="font-weight: 300;">2) Запускаем postgresql-server: </span><span style="font-weight: 300;"><code>systemctl start postgresql</code></span></p>
<p><span style="font-weight: 300;">3) Добавляем postgresql-server в автозагрузку:&nbsp; </span><span style="font-weight: 300;"><code>systemctl enable postgresql</code></span></p>
<p><span style="font-weight: 300;">Настраивать резервное копирование мы будем с помощью утилиты Barman. В документации Barman рекомендуется разворачивать Barman на отдельном сервере. В этом случае потребуется настроить доступы между серверами по SSH-ключам. В данном задании мы будем разворачивать Barman на отдельном хосте barman.</span></p>
<p><span style="font-weight: 300;">На хостах node1 и node2 необходимо установить утилиту barman-cli: <code>apt install barman-cli</code></span></p>
<p><span style="font-weight: 300;">На хосте barman устанавливаем пакеты barman и postgresql-client: <code>apt install barman-cli barman postgresql</code></span></p>
<p><span style="font-weight: 300;">Все это сделаем с помощью Ansible. В Vagrant-файле есть блок для запуска плейбука после развертывания ВМ. В этом же каталоге создаем плейбук provision.yml и файл hosts со списком хостов. В плейбук добавляем установку ПО и запуск служб. Файлы прикладываю сюда.</span></p>
<p><span style="font-weight: 300;">В результате выполнения команды <code>vagrant up</code> получим 3 ВМ с установленным ПО.</span></p>
<img width="1095" height="345" alt="image" src="https://github.com/user-attachments/assets/21ee694a-ec85-49dd-9c7e-633312dfd93c" />
<p>&nbsp;</p>
<p><span style="font-weight: 300;">Стенд собран, можно начинать.</span></p>
<p><strong>Настройка hot_standby репликации с использованием слотов</strong></p>
<p>На хосте node1:&nbsp;</p>
<p><span style="font-weight: 300;">1) Заходим в psql:</span></p>
<p><code>sudo -u postgres psql</code></p>
<img width="762" height="798" alt="image" src="https://github.com/user-attachments/assets/f1a10f4b-1a69-48d1-a97a-c1dd1220c766" />
<p>&nbsp;</p>
<p><span style="font-weight: 300;">2) В psql создаём пользователя replicator c правами репликации и паролем &laquo;Otus2022!&raquo;</span></p>
<p><span style="font-weight: 300;"><code>CREATE USER replicator WITH REPLICATION Encrypted PASSWORD 'Otus2022!';</code></span></p>
<img width="862" height="84" alt="image" src="https://github.com/user-attachments/assets/2fc3cd7f-67e5-4d9a-bf3c-36236d0ac2ce" />
<p>&nbsp;</p>
<p><span style="font-weight: 300;">3) В файле </span><span style="font-weight: 400;">/etc/postgresql/14/main/postgresql.conf</span> <span style="font-weight: 300;">указываем следующие параметры:</span></p>
<p><span style="font-weight: 400;">#Указываем ip-адреса, на которых postgres будет слушать трафик на порту 5432 (параметр port)</span></p>
<p><span style="font-weight: 300;">listen_addresses = 'localhost, 192.168.57.11'</span></p>
<p><span style="font-weight: 400;">#Указываем порт порт postgres</span></p>
<p><span style="font-weight: 300;">port = 5432&nbsp;</span></p>
<p><span style="font-weight: 400;">#Устанавливаем максимально 100 одновременных подключений</span></p>
<p><span style="font-weight: 300;">max_connections = 100</span></p>
<p><span style="font-weight: 300;">log_directory = 'log'&nbsp;</span></p>
<p><span style="font-weight: 300;">log_filename = 'postgresql-%a.log'&nbsp;</span></p>
<p><span style="font-weight: 300;">log_rotation_age = 1d&nbsp;</span></p>
<p><span style="font-weight: 300;">log_rotation_size = 0&nbsp;</span></p>
<p><span style="font-weight: 300;">log_truncate_on_rotation = on&nbsp;</span></p>
<p><span style="font-weight: 300;">max_wal_size = 1GB</span></p>
<p><span style="font-weight: 300;">min_wal_size = 80MB</span></p>
<p><span style="font-weight: 300;">log_line_prefix = '%m [%p] '&nbsp;</span></p>
<p><span style="font-weight: 400;">#Указываем часовой пояс для Москвы</span></p>
<p><span style="font-weight: 300;">log_timezone = 'UTC+3'</span></p>
<p><span style="font-weight: 300;">timezone = 'UTC+3'</span></p>
<p><span style="font-weight: 300;">datestyle = 'iso, mdy'</span></p>
<p><span style="font-weight: 300;">lc_messages = 'en_US.UTF-8'</span></p>
<p><span style="font-weight: 300;">lc_monetary = 'en_US.UTF-8'&nbsp;</span></p>
<p><span style="font-weight: 300;">lc_numeric = 'en_US.UTF-8'&nbsp;</span></p>
<p><span style="font-weight: 300;">lc_time = 'en_US.UTF-8'&nbsp;</span></p>
<p><span style="font-weight: 300;">default_text_search_config = 'pg_catalog.english'</span></p>
<p><span style="font-weight: 400;">#можно или нет подключаться к postgresql для выполнения запросов в процессе восстановления;&nbsp;</span></p>
<p><span style="font-weight: 300;">hot_standby = on</span></p>
<p><span style="font-weight: 400;">#Включаем репликацию</span></p>
<p><span style="font-weight: 300;">wal_level = replica</span></p>
<p><span style="font-weight: 400;">#Количество планируемых слейвов</span></p>
<p><span style="font-weight: 300;">max_wal_senders = 3</span></p>
<p><span style="font-weight: 400;">#Максимальное количество слотов репликации</span></p>
<p><span style="font-weight: 300;">max_replication_slots = 3</span></p>
<p><span style="font-weight: 400;">#будет ли сервер slave&nbsp;сообщать мастеру&nbsp;о запросах, которые он выполняет.</span></p>
<p><span style="font-weight: 300;">hot_standby_feedback = on</span></p>
<p><span style="font-weight: 400;">#Включаем использование зашифрованных паролей</span></p>
<p><span style="font-weight: 300;">password_encryption = scram-sha-256</span></p>
<p><span style="font-weight: 300;">4) Настраиваем параметры подключения в файле </span><span style="font-weight: 400;">/etc/postgresql/14/main/pg_hba.conf:&nbsp;</span></p>
<p><span style="font-weight: 400;"># TYPE&nbsp; DATABASE&nbsp; &nbsp; &nbsp; &nbsp; USER&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ADDRESS &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; METHOD</span></p>
<p><span style="font-weight: 400;"># "local" is for Unix domain socket connections only</span></p>
<p><span style="font-weight: 400;">local &nbsp; all&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; all&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; peer</span></p>
<p><span style="font-weight: 400;"># IPv4 local connections:</span></p>
<p><span style="font-weight: 400;">host&nbsp; &nbsp; all&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; all &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 127.0</span><strong>.</strong><span style="font-weight: 400;">0.1</span><strong>/</strong><span style="font-weight: 400;">32&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; scram</span><strong>-</strong><span style="font-weight: 400;">sha</span><strong>-</strong><span style="font-weight: 400;">256</span></p>
<p><span style="font-weight: 400;"># IPv6 local connections:</span></p>
<p><span style="font-weight: 400;">host&nbsp; &nbsp; all&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; all &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; </span><strong>::</strong><span style="font-weight: 400;">1</span><strong>/</strong><span style="font-weight: 400;">128 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; scram</span><strong>-</strong><span style="font-weight: 400;">sha</span><strong>-</strong><span style="font-weight: 400;">256</span></p>
<p><span style="font-weight: 400;"># Allow replication connections from localhost, by a user with the</span></p>
<p><span style="font-weight: 400;"># replication privilege.</span></p>
<p><span style="font-weight: 400;">local &nbsp; replication&nbsp; &nbsp; &nbsp; all&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; peer</span></p>
<p><span style="font-weight: 400;">host&nbsp; &nbsp; replication &nbsp; &nbsp; all &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 127.0</span><strong>.</strong><span style="font-weight: 400;">0.1</span><strong>/</strong><span style="font-weight: 400;">32 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; scram</span><strong>-</strong><span style="font-weight: 400;">sha</span><strong>-</strong><span style="font-weight: 400;">256</span></p>
<p><span style="font-weight: 400;">host&nbsp; &nbsp; replication &nbsp; &nbsp; all &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; </span><strong>::</strong><span style="font-weight: 400;">1</span><strong>/</strong><span style="font-weight: 400;">128&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; scram</span><strong>-</strong><span style="font-weight: 400;">sha</span><strong>-</strong><span style="font-weight: 400;">256</span></p>
<p><strong>host&nbsp; &nbsp; replication replicator&nbsp; &nbsp; 192.168.57.11/32&nbsp; &nbsp; &nbsp; &nbsp; scram-sha-256</strong></p>
<p><strong>host&nbsp; &nbsp; replication replicator&nbsp; &nbsp; 192.168.57.12/32&nbsp; &nbsp; &nbsp; &nbsp; scram-sha-256</strong></p>
<p><span style="font-weight: 300;">Две последние строки в файле разрешают репликацию пользователю replicator.&nbsp;</span></p>
<p><span style="font-weight: 300;">5) Перезапускаем postgresql-server: </span><span style="font-weight: 300;"><code>systemctl restart postgresql</code></span></p>
<p><span style="font-weight: 400;">На хосте node2:&nbsp;</span></p>
<p><span style="font-weight: 300;">1) Останавливаем postgresql-server: </span><span style="font-weight: 300;"><code>systemctl stop postgresql</code></span></p>
<p><span style="font-weight: 300;">2) С помощью утилиты pg_basebackup копируем данные с node1:</span></p>
<p><span style="font-weight: 300;"><code>pg_basebackup -h 192.168.57.11 -U replicator -D /var/lib/postgresql/14/main -R -P</code></span></p>
<p><span style="font-weight: 300;">*** Будет ошибка, что каталог /var/lib/postgresql/14/main не пуст. Надо удалить все из этого каталога, а затем запустить команду повторно.</span></p>
<p><span style="font-weight: 300;">3) В файле&nbsp; </span><span style="font-weight: 400;">/etc/postgresql/14/main/postgresql.conf </span><span style="font-weight: 300;">меняем параметр:</span></p>
<p><span style="font-weight: 300;">listen_addresses = 'localhost, </span><span style="font-weight: 400;">192.168.57.12'</span><span style="font-weight: 300;">&nbsp;</span></p>
<p><span style="font-weight: 300;">4) Запускаем службу postgresql-server: </span><span style="font-weight: 300;"><code>systemctl start postgresql</code></span></p>
<p><span style="font-weight: 300;">*** Служба не запускается, ошибка доступа. Надо запустить команду <code>sudo chown -R postgres:postgres /var/lib/postgresql/14/main</code> и снова запустить службу.</span></p>
<p><span style="font-weight: 300;">Проверка репликации:&nbsp;</span></p>
<p><span style="font-weight: 300;">На хосте node1 в psql создадим базу otus_test и выведем список БД:&nbsp;</span></p>
<p><span style="font-weight: 300;">postgres</span><span style="font-weight: 400;">=</span><span style="font-weight: 300;"># </span><span style="font-weight: 400;"><code>CREATE DATABASE otus_test;</code></span></p>
<p><span style="font-weight: 300;">postgres</span><span style="font-weight: 400;">=</span><span style="font-weight: 300;"># </span><span style="font-weight: 400;"><code>\l</code></span></p>
<img width="862" height="449" alt="image" src="https://github.com/user-attachments/assets/92bd5812-bab5-444f-89f5-49cf4d3c6563" />
<p>&nbsp;</p>
<p><span style="font-weight: 300;">На хосте node2 также в psql также проверим список БД (команда </span><span style="font-weight: 400;"><code>\l</code></span><span style="font-weight: 300;">), в списке БД должна появится БД </span><span style="font-weight: 400;">otus_test.</span><span style="font-weight: 300;">&nbsp;</span></p>
<img width="1299" height="407" alt="image" src="https://github.com/user-attachments/assets/aa401e10-cd6d-4ee4-83cf-db193d6744d2" />
<p>&nbsp;</p>
<p><span style="font-weight: 300;">Также можно проверить репликацию другим способом:&nbsp;</span></p>
<p><span style="font-weight: 300;">На хосте node1 в psql вводим команду: </span><span style="font-weight: 300;"><code>select * from pg_stat_replication;</code></span></p>
<p><span style="font-weight: 300;">На хосте node2 в psql вводим команду: </span><span style="font-weight: 300;"><code>select * from pg_stat_wal_receiver;</code></span></p>
<p><span style="font-weight: 300;">Вывод обеих команд должен быть </span><span style="font-weight: 300;">не пустым.</span><span style="font-weight: 300;">&nbsp;</span></p>
<img width="1852" height="219" alt="image" src="https://github.com/user-attachments/assets/0d77bd75-bc87-4f91-b574-f6cb002f81cc" />
<img width="1852" height="267" alt="image" src="https://github.com/user-attachments/assets/82f8daee-fc22-4844-bb69-7f4303c4cd9c" />
<p>&nbsp;</p>
<p><span style="font-weight: 300;">На этом настройка репликации завершена. </span></p>










