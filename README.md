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
<p><strong>Настройка резервного копирования</strong></p>
<p><span style="font-weight: 300;">На хосте barman выполняем следующие настройки:&nbsp;</span></p>
<p><span style="font-weight: 300;">Ставим пакет postgresql-client <code>apt install postgresql-client</code></span></p>
<p><span style="font-weight: 300;">Переходим в пользователя barman и генерируем ssh-ключ:&nbsp;</span></p>
<p><span style="font-weight: 300;"><code>su barman</code></span></p>
<p><span style="font-weight: 300;"><code>cd</code></span></p>
<p><span style="font-weight: 300;"><code>ssh-keygen -t rsa -b 4096</code></span></p>
<img width="709" height="554" alt="image" src="https://github.com/user-attachments/assets/72796037-5803-4f3d-9ea5-956f6950f184" />
<p>&nbsp;</p>
<p><span style="font-weight: 300;">На хосте node1:</span></p>
<p><span style="font-weight: 300;">Переходим в пользователя postgres и генерируем ssh-ключ:</span></p>
<p><span style="font-weight: 300;"><code>su postgres</code></span></p>
<p><span style="font-weight: 300;"><code>cd</code></span></p>
<p><span style="font-weight: 300;"><code>ssh-keygen -t rsa -b 4096</code></span></p>
<img width="709" height="561" alt="image" src="https://github.com/user-attachments/assets/338ff5eb-1607-4371-b998-1f3fc2d6b592" />
<p>&nbsp;</p>
<p><span style="font-weight: 300;">После генерации ключа, выводим содержимое файла ~/.ssh/id_rsa.pub:&nbsp;</span></p>
<p><span style="font-weight: 300;"><code>cat ~/.ssh/id_rsa.pub</code></span></p>
<p><span style="font-weight: 300;">Копируем содержимое файла на сервер </span><span style="font-weight: 400;">barman</span><span style="font-weight: 300;"> в файл </span><span style="font-weight: 400;">/var/lib/barman/.ssh/authorized_keys</span></p>
<p><span style="font-weight: 300;">В psql создаём пользователя barman c правами суперпользователя:</span></p>
<p><span style="font-weight: 300;"><code>CREATE USER barman WITH REPLICATION Encrypted PASSWORD 'Otus2022!';</code></span></p>
<p><span style="font-weight: 300;">В файл /etc/postgresql/14/main/pg_hba.conf добавляем разрешения для пользователя barman:</span></p>
<p><span style="font-weight: 300;"># TYPE&nbsp; DATABASE&nbsp; &nbsp; &nbsp; &nbsp; USER&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ADDRESS &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; METHOD</span></p>
<p><span style="font-weight: 300;"># "local" is for Unix domain socket connections only</span></p>
<p><span style="font-weight: 300;">local &nbsp; all &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; all &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; peer</span></p>
<p><span style="font-weight: 300;"># IPv4 local connections:</span></p>
<p><span style="font-weight: 300;">host&nbsp; &nbsp; all &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; all &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 127.0.0.1/32&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; scram-sha-256</span></p>
<p><span style="font-weight: 300;"># IPv6 local connections:</span></p>
<p><span style="font-weight: 300;">host&nbsp; &nbsp; all &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; all &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ::1/128 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; scram-sha-256</span></p>
<p><span style="font-weight: 300;"># Allow replication connections from localhost, by a user with the</span></p>
<p><span style="font-weight: 300;"># replication privilege.</span></p>
<p><span style="font-weight: 300;">local &nbsp; replication &nbsp; &nbsp; all &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; peer</span></p>
<p><span style="font-weight: 300;">host&nbsp; &nbsp; replication &nbsp; &nbsp; all &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 127.0.0.1/32 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; scram-sha-256</span></p>
<p><span style="font-weight: 300;">host&nbsp; &nbsp; replication &nbsp; &nbsp; all &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ::1/128&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; scram-sha-256</span></p>
<p><span style="font-weight: 300;">host&nbsp; &nbsp; replication replicator &nbsp; 192.168.57.11/32&nbsp; &nbsp; &nbsp; &nbsp; scram-sha-256</span></p>
<p><span style="font-weight: 300;">host&nbsp; &nbsp; replication replicator &nbsp; 192.168.57.12/32&nbsp; &nbsp; &nbsp; &nbsp; scram-sha-256</span></p>
<p><span style="font-weight: 400;"><strong>host&nbsp; &nbsp; all &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; barman &nbsp; &nbsp; &nbsp; 192.168.57.13/32&nbsp; &nbsp; &nbsp; &nbsp; scram-sha-256</strong></span></p>
<p><span style="font-weight: 400;"><strong>host&nbsp; &nbsp; replication &nbsp; barman &nbsp; &nbsp; &nbsp; 192.168.57.13/32&nbsp; &nbsp; &nbsp; scram-sha-256</strong></span></p>
<p><span style="font-weight: 300;">Перезапускаем службу postgresql-14: </span><span style="font-weight: 300;"><code>systemctl restart postgresql</code></span></p>
<p><span style="font-weight: 300;">В psql создадим тестовую базу otus: </span><span style="font-weight: 300;"><code>CREATE DATABASE otus;</code></span></p>
<p><span style="font-weight: 300;">В базе создаём таблицу test в базе otus:</span></p>
<p><span style="font-weight: 300;"><code>\c otus;</code></span></p>
<p><span style="font-weight: 300;"><code>CREATE TABLE test (id int, name varchar(30));</code></span></p>
<p><span style="font-weight: 300;"><code>INSERT INTO test VALUES (1, 'alex');</code></span></p>
<p><span style="font-weight: 300;">Видим таблицу test с содержимым:</span></p>
<img width="709" height="473" alt="image" src="https://github.com/user-attachments/assets/9b5d5f22-b4ad-4eb0-8968-e0666511cbf6" />
<p>&nbsp;</p>
<p><span style="font-weight: 400;">На хосте barman:</span></p>
<p><span style="font-weight: 300;">После генерации ключа, выводим содержимое файла ~/.ssh/id_rsa.pub:</span></p>
<p><span style="font-weight: 300;"><code>cat ~/.ssh/id_rsa.pub</code></span></p>
<p><span style="font-weight: 300;">Копируем содержимое файла на сервер </span><span style="font-weight: 400;">postgres</span><span style="font-weight: 300;"> в файл </span><span style="font-weight: 400;">/var/lib/postgresql/.ssh/authorized_keys</span></p>
<p>Находясь в пользователе barman, создаём файл ~/.pgpass со следующим содержимым:</p>
<p><span style="font-weight: 300;">192.168.57.11:5432:*:barman:Otus2022!</span></p>
<p><span style="font-weight: 300;">В данном файле указываются реквизиты доступа для postgres. Через знак двоеточия пишутся следующие параметры:&nbsp;</span></p>
<ul>
<li style="font-weight: 300;"><span style="font-weight: 300;">ip-адрес</span></li>
<li style="font-weight: 300;"><span style="font-weight: 300;">порт postgres</span></li>
<li style="font-weight: 300;"><span style="font-weight: 300;">имя БД (* означает подключение к любой БД)</span></li>
<li style="font-weight: 300;"><span style="font-weight: 300;">имя пользователя</span></li>
<li style="font-weight: 300;"><span style="font-weight: 300;">пароль пользователя</span></li>
</ul>
<p><span style="font-weight: 300;">Файл должен быть с правами 600, владелец файла barman. </span></p>
<img width="418" height="146" alt="image" src="https://github.com/user-attachments/assets/5c4c89ba-d331-49c4-b2b5-be94a36e8b7d" />
<p>&nbsp;</p>
<p><span style="font-weight: 300;">После создания postgres-пользователя barman необходимо проверить, что права для пользователя настроены корректно:&nbsp;</span></p>
<p><span style="font-weight: 300;">Проверяем возможность подключения к postgres-серверу:&nbsp;</span></p>
<p><span style="font-weight: 300;"><code>psql -h 192.168.57.11 -U barman -d postgres</code></span></p>
<img width="973" height="149" alt="image" src="https://github.com/user-attachments/assets/dae3d86d-f71b-4e36-bb30-c18dc162a91e" />
<p>&nbsp;</p>
<p><span style="font-weight: 300;">Проверяем репликацию:</span></p>
<p><span style="font-weight: 300;"><code>psql -h 192.168.57.11 -U barman -c "IDENTIFY_SYSTEM" replication=1</code></span></p>
<img width="854" height="167" alt="image" src="https://github.com/user-attachments/assets/3ab75988-478c-4484-b05a-2066922fbcda" />
<p>&nbsp;</p>
<p><span style="font-weight: 300;">Создаём файл </span><span style="font-weight: 300;">/etc/barman.conf </span><span style="font-weight: 300;">со следующим содержимым&nbsp;</span></p>
<p><span style="font-weight: 300;">Владельцем файла должен быть пользователь barman.</span></p>
<p><span style="font-weight: 300;">[barman]</span></p>
<p><span style="font-weight: 400;">#Указываем каталог, в котором будут храниться бекапы</span></p>
<p><span style="font-weight: 300;">barman_home = /var/lib/barman</span></p>
<p><span style="font-weight: 400;">#Указываем каталог, в котором будут храниться файлы конфигурации бекапов</span></p>
<p><span style="font-weight: 300;">configuration_files_directory = /etc/barman.d</span></p>
<p><span style="font-weight: 400;">#пользователь, от которого будет запускаться barman</span></p>
<p><span style="font-weight: 300;">barman_user = barman</span></p>
<p><span style="font-weight: 400;">#расположение файла с логами</span></p>
<p><span style="font-weight: 300;">log_file = /var/log/barman/barman.log</span></p>
<p><span style="font-weight: 400;">#Используемый тип сжатия</span></p>
<p><span style="font-weight: 300;">compression = gzip</span></p>
<p><span style="font-weight: 400;">#Используемый метод бекапа</span></p>
<p><span style="font-weight: 300;">backup_method = rsync</span></p>
<p><span style="font-weight: 300;">archiver = on</span></p>
<p><span style="font-weight: 300;">retention_policy = REDUNDANCY 3</span></p>
<p><span style="font-weight: 300;">immediate_checkpoint = true</span></p>
<p><span style="font-weight: 400;">#Глубина архива</span></p>
<p><span style="font-weight: 300;">last_backup_maximum_age = 4 DAYS</span></p>
<p><span style="font-weight: 300;">minimum_redundancy = 1</span></p>
<p><em><span style="font-weight: 300;">&nbsp;</span></em><span style="font-weight: 300;">Создаём файл </span><span style="font-weight: 300;">/etc/barman.d/node1.conf </span><span style="font-weight: 300;">со следующим содержимым&nbsp;</span></p>
<p><span style="font-weight: 300;">Владельцем файла должен быть пользователь barman.</span></p>
<p><span style="font-weight: 300;">[node1]</span></p>
<p><span style="font-weight: 400;">#Описание задания</span></p>
<p><span style="font-weight: 300;">description = "backup node1"</span></p>
<p><span style="font-weight: 400;">#Команда подключения к хосту node1</span></p>
<p><span style="font-weight: 300;">ssh_command = ssh postgres@192.168.57.11&nbsp;</span></p>
<p><span style="font-weight: 400;">#Команда для подключения к postgres-серверу</span></p>
<p><span style="font-weight: 300;">conninfo = host=192.168.57.11 user=barman port=5432 dbname=postgres</span></p>
<p><span style="font-weight: 300;">retention_policy_mode = auto</span></p>
<p><span style="font-weight: 300;">retention_policy = RECOVERY WINDOW OF 7 days</span></p>
<p><span style="font-weight: 300;">wal_retention_policy = main</span></p>
<p><span style="font-weight: 300;">streaming_archiver=on</span></p>
<p><span style="font-weight: 400;">#Указание префикса, который будет использоваться как $PATH на хосте node1</span></p>
<p><span style="font-weight: 300;">path_prefix = /usr/pgsql-14/bin/</span></p>
<p><span style="font-weight: 400;">#настройки слота</span></p>
<p><span style="font-weight: 300;">create_slot = auto</span></p>
<p><span style="font-weight: 300;">slot_name = node1</span></p>
<p><span style="font-weight: 400;">#Команда для потоковой передачи от postgres-сервера</span></p>
<p><span style="font-weight: 300;">streaming_conninfo = host=192.168.57.11 user=barman&nbsp;</span></p>
<p><span style="font-weight: 400;">#Тип выполняемого бекапа</span></p>
<p><span style="font-weight: 300;">backup_method = postgres</span></p>
<p><span style="font-weight: 300;">archiver = off</span></p>
<p><span style="font-weight: 300;">На этом настройка бекапа завершена. Теперь проверим работу barman:&nbsp;</span></p>
<p><span style="font-weight: 400;"><code>barman switch-wal node1</code></span></p>
<img width="1128" height="74" alt="image" src="https://github.com/user-attachments/assets/c864168e-a937-43b0-8217-8006b92fbad0" />
<p>&nbsp;</p>
<p><span style="font-weight: 400;">Видим ошибку. Пользователю barman не хватает привилегий на сервере postgres. Нужно ему их дать. На сервере node1 выдаем привилегии:</span></p>
<p>postgres=# <code>GRANT pg_read_all_settings TO barman;</code></p>
<p>postgres=# <code>GRANT pg_read_all_stats TO barman;</code></p>
<p>postgres=# <code>GRANT EXECUTE ON FUNCTION pg_start_backup(text, boolean, boolean) TO barman;</code></p>
<p>postgres=# <code>GRANT EXECUTE ON FUNCTION pg_stop_backup() TO barman;</code></p>
<p>postgres=# <code>GRANT EXECUTE ON FUNCTION pg_switch_wal() TO barman;</code></p>
<p>postgres=# <code>GRANT EXECUTE ON FUNCTION pg_create_restore_point(text) TO barman;</code></p>
<p><span style="font-weight: 400;">После этого команда выполняется без ошибок:</span></p>
<img width="801" height="74" alt="image" src="https://github.com/user-attachments/assets/05877ee2-b21c-4ca2-8065-4c174bc508b5" />
<p>&nbsp;</p>
<p><span style="font-weight: 400;">Далее выполним команды <code>barman cron</code> и <code>barman check node1</code></span></p>
<img width="801" height="76" alt="image" src="https://github.com/user-attachments/assets/19d7ba6a-e8c3-4938-8b91-087f012b2340" />
<img width="894" height="561" alt="image" src="https://github.com/user-attachments/assets/24e8e181-945b-4900-955b-2eb41acd8a66" />
<p>&nbsp;</p>
<p><span style="font-weight: 300;">Если во всех пунктах OK, значит бекап отработает корректно.</span></p>
<p><span style="font-weight: 300;">После этого запускаем резервную копию:</span></p>
<p><span style="font-weight: 400;"><code>barman backup node1</code></span></p>
<img width="1652" height="520" alt="image" src="https://github.com/user-attachments/assets/aee64e98-780a-4450-8b01-96ab3a4f1043" />
<p>&nbsp;</p>
<p><span style="font-weight: 300;">На этом процесс настройки бекапа закончен, для удобства команду <code>barman backup node1</code> требуется добавить в crontab.</span></p>
<p><span style="font-weight: 400;">Проверка восстановления из бекапов:</span></p>
<p><span style="font-weight: 300;">На хосте node1 в psql удаляем базы Otus:</span></p>
<p><span style="font-weight: 300;">postgres=# </span><span style="font-weight: 400;"><code>DROP DATABASE otus;</code></span></p>
<p><span style="font-weight: 300;">postgres=# </span><span style="font-weight: 400;"><code>DROP DATABASE otus_test;</code></span></p>
<img width="663" height="228" alt="image" src="https://github.com/user-attachments/assets/8d2528fb-3efe-42a5-9b9b-b6862d56e044" />
<p>&nbsp;</p>
<p><span style="font-weight: 300;">Далее на хосте barman запустим восстановление:</span></p>
<p><span style="font-weight: 400;"><code>barman list-backup node1</code></span></p>
<img width="895" height="100" alt="image" src="https://github.com/user-attachments/assets/483ef28c-5b34-4e42-b3a2-9a1d710b4100" />
<p>&nbsp;</p>
<p><span style="font-weight: 300;">Видим два бекапа (после первого я еще раз запускал). Запустим восстановление из последнего:</span></p>
<p><span style="font-weight: 400;"><code>barman recover node1 20260109T183558 /var/lib/postgresql/14/main/ --remote-ssh-comman "ssh postgres@192.168.57.11"</code></span></p>
<img width="1329" height="560" alt="image" src="https://github.com/user-attachments/assets/9d5b46a1-fe52-41f0-b744-6a05c7f83356" />
<p>&nbsp;</p>
<p><span style="font-weight: 300;">Далее на хосте node1 потребуется перезапустить postgresql-сервер и снова проверить список БД. Базы otus должны вернуться обратно.</span></p>
<img width="854" height="775" alt="image" src="https://github.com/user-attachments/assets/aaed2fd2-7448-400e-977b-7c91f9638fc6" />
<p>&nbsp;</p>
<p><span style="font-weight: 300;">Бекап успешно восстановлен. Задание завершено.</span></p>












