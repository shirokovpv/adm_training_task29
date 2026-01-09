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
<p><span style="font-weight: 300;">apt install -y telnet</span></p>
<p><span style="font-weight: 300;">Команды должны выполняться от root-пользователя</span></p>
<p><span style="font-weight: 300;">Для перехода в root-пользователя вводим </span><span style="font-weight: 300;">sudo -i</span></p>
<p><span style="font-weight: 300;">Перед настройкой репликации необходимо установить postgres-server на хосты node1 и node2:</span></p>
<p><span style="font-weight: 300;">1) Устанавливаем postgresql-server 14: </span><span style="font-weight: 300;">apt install postgresql postgresql-contrib</span></p>
<p><span style="font-weight: 300;">2) Запускаем postgresql-server: </span><span style="font-weight: 300;">systemctl start postgresql</span></p>
<p><span style="font-weight: 300;">3) Добавляем postgresql-server в автозагрузку:&nbsp; </span><span style="font-weight: 300;">systemctl enable postgresql</span></p>
<p><span style="font-weight: 300;">Настраивать резервное копирование мы будем с помощью утилиты Barman. В документации Barman рекомендуется разворачивать Barman на отдельном сервере. В этом случае потребуется настроить доступы между серверами по SSH-ключам. В данном задании мы будем разворачивать Barman на отдельном хосте barman.</span></p>
<p><span style="font-weight: 300;">На хостах node1 и node2 необходимо установить утилиту barman-cli: apt install barman-cli</span></p>
<p><span style="font-weight: 300;">На хосте barman устанавливаем пакеты barman и postgresql-client: </span><span style="font-weight: 300;">apt install barman-cli barman postgresql</span></p>
<p><span style="font-weight: 300;">Все это сделаем с помощью Ansible. В Vagrant-файле есть блок для запуска плейбука после развертывания ВМ. В этом же каталоге создаем плейбук provision.yml и файл hosts со списком хостов. В плейбук добавляем установку ПО и запуск служб. Файлы прикладываю сюда.</span></p>






