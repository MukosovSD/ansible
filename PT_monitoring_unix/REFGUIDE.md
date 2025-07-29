## Linux-подобные ОС: настройка источников

Настройку источника нужно выполнять от имени учетной записи root.

```
При использовании в IT-инфраструктуре организации межсетевого экрана или других средств для контроля сетевого трафика
требуется настроить в них правила, разрешающие трафик между узлом источника и узлом MP 10 Agent через порты UDP 514 или
TCP 1468 (в зависимости от используемого протокола) в направлении узла MP 10 Agent.
```

Поддерживается сбор событий со следующих ОС семейства Linux:
- ALT Linux 8;
- Astra Linux Special Edition 1.6, 1.7;
- Canonical Ubuntu 18.04, 20.04, 22.04;
- CentOS 7, 8;
- Debian 9 - 11;
- Oracle Linux 7, 8;
- Red Hat Enterprise Linux 7, 8;
- SUSE Linux Enterprise Server 12, 15;
- РЕД ОС 7.1 - 7.3.

Для журналирования событий в ОС используются служба auditd и служба журналирования rsyslog, syslog-ng или syslogd.

Предусмотрены несколько методов настройки источников: автоматическая настройка с помощью роли для системы управления
конфигурациями Ansible, автоматическая настройка с помощью сценария на языке Python или ручная настройка.

<br/>

**Настройка с помощью роли для Ansible**

```
Настройка источников с помощью роли Ansible поддерживается для Astra Linux, Canonical Ubuntu, CentOS, Debian,
Oracle Linux, Red Hat Enterprise Linux и РЕД ОС.
```

Для настройки источника нужно:
1. Создать учетную запись для удаленной настройки с правом повышения привилегий с помощью su или sudo.
   ```
   На всех настраиваемых источниках рекомендуется создать учетные записи с одинаковыми логином и 
   паролем.
   ```
2. Применить роль Ansible.
3. Проверить настройку службы auditd.
4. Определить используемую на источнике службу журналирования.
5. Если используется служба syslogd — настроить ее.

<br/>

**Настройка с помощью сценария**

Для настройки источника нужно:
1. Установить или обновить службу auditd.
2. Определить используемую на источнике службу журналирования.
3. Выполнить сценарий настройки источника на языке Python.
4. Проверить настройку службы auditd.
5. Если используется служба журналирования syslog-ng — настроить ее.

<br/>

**Настройка вручную**

Для настройки источника нужно:
1. Установить или обновить службу auditd.
2. Настроить адресацию и формат событий.
3. Настроить правила журналирования.
4. Проверить настройку службы auditd.
5. Определить используемую на источнике службу журналирования.
6. Настроить используемую службу.
7. Настроить отправку событий syslog через audispd.

<br/>


### Применение роли Ansible

```
Для выполнения инструкции требуется система управления конфигурациями Ansible версии 2.9 или выше.
```

При применении роли на источниках автоматически настраиваются формат событий, правила журналирования, служба
журналирования (rsyslog или syslog-ng) и отправка событий syslog через audispd.

<br/>

**Удаленное применение роли**

► Чтобы удаленно применить роль Ansible для нескольких источников:

1. Скачайте архив с файлами роли `lep-src-ansible-role.tar`.

   ```
   Вы можете скачать архив с файлами роли из хранилища файлов.
   ```

2. Откройте файл `roles/lep-src/vars/siem_agents.yml`.

3. В секцию `facilities` добавьте строки с произвольными псевдонимами для MP 10 Agent, на которые будут отправляться
   события от различных источников. По умолчанию добавлена секция с псевдонимом `default`.

4. В каждую секцию с псевдонимом добавьте строку с IP-адресом или полными доменным именем (FQDN) узла MP 10 Agent,
   транспортным протоколом (`udp` или `tcp`) и номером порта подключения (`514` или `1468`):

   ```yaml
   <Псевдоним>:
       - { address: '<Адрес MP 10 Agent>', transport: '<Протокол>', port: '<Порт>' }
   ```

   ```
   Вы можете добавить несколько строк с адресами узлов MP 10 Agent в одну секцию с псевдонимом. В этом случае события 
   отправляются одновременно на все указанные узлы.
   ```

5. Сохраните изменения и закройте файл.

6. Откройте файл `hosts`.

7. В параметре `ansible_ssh_user` укажите логин учетной записи для удаленной настройки источников.

8. Добавьте строки с IP-адресами или полными доменными именами (FQDN) источников, которые необходимо настроить:

   ```
   <Адрес> facility=<Псевдоним>
   ```

   Если для источника не указан псевдоним, события отправляются на узел, указанный в секции с
   псевдонимом `default`.

9. Сохраните изменения и закройте файл.

   Вы можете изменить другие параметры для настройки источников в файле `roles/lep-src/vars/main.yml`
   (см. таблицу ниже). Если требуется изменить параметры отдельного источника, вы также можете добавить в файл
   `hosts` строки вида `<IP-адрес источника> <Параметр>=<Значение>`. Дополнительная информация о применении роли
    доступна на сайте [docs.ansible.com](https://docs.ansible.com/ansible/latest/).

10. Выполните команду для применения роли:

    ```bash
    ansible-playbook -i hosts --ask-become-pass -k ./lep_src_ansible.yml
    ```

11. Введите пароль учетной записи для удаленной настройки.

<br/>

**Локальное применение роли**

► Чтобы локально применить роль Ansible на источнике:

1. Скачайте архив с файлами роли `lep-src-ansible-role.tar`.

   ```
   Вы можете скачать архив с файлами роли из хранилища файлов.
   ```

2. Откройте файл `roles/lep-src/vars/siem_agents.yml`.

3. В секцию `facilities` → `default` добавьте строку с IP-адресом или полными доменным именем (FQDN) узла MP 10 Agent,
   транспортным протоколом (`udp` или `tcp`) и номером порта подключения (`514` или `1468`):

   ```yaml
   default:
       - { address: '<Адрес MP 10 Agent>', transport: '<Протокол>', port: '<Порт>' }
   ```

   ```
   Вы можете добавить несколько строк с адресами узлов MP 10 Agent. В этом случае события отправляются одновременно
   на все указанные узлы.
   ```

4. Сохраните изменения и закройте файл.

   ```
   Вы можете изменить другие параметры для настройки источника в файле /roles/lep-src/vars/main.yml (см. таблицу ниже).
   ```

5. Выполните команду для применения роли:

   ```bash
   ansible-playbook --connection=local --ask-become-pass ./local_lep_src_ansible.yml
   ```

6. Введите пароль учетной записи (с правом повышения привилегий с помощью su или sudo).


<table>
  <caption>Параметры для настройки источников (для файла main.yml)</caption>
  <colgroup><col style="width: 30.3%;"/><col style="width: 69.6%;"/></colgroup>
<thead><tr>
  <th><span style="text-align:left; font-weight:bold;">Параметр</span></th>
  <th><span style="text-align:left; font-weight:bold;">Описание</span></th>
</tr></thead>

<tbody>

<tr><td style="vertical-align:top;" colspan="2">
  <span style="emphasis; font-weight:bold;">
    Параметры службы auditd
  </span>
</td></tr>

<tr><td style="vertical-align:top;">
  <span style="text-align:left">

  `audispd_plugin_config_v2`

  </span>
</td>
<td style="vertical-align:top;">
  <span style="text-align:left">

  Расположение конфигурационного файла плагина auditsp для службы auditd версии 2. 
  По умолчанию `/etc/audisp/plugins.d/syslog.conf`
  
  </span>
</td></tr>

<tr><td style="vertical-align:top;">
  <span style="text-align:left">

  `audispd_plugin_config_v3`

  </span>
</td><td style="vertical-align:top;">
  <span style="text-align:left">

  Расположение конфигурационного файла плагина auditsp для службы auditd версии 3. 
  По умолчанию `/etc/audit/plugins.d/syslog.conf`

  </span>
</td></tr>

<tr><td style="vertical-align:top;">
  <span style="text-align:left">

  `auditd_config`

  </span>
</td>
<td style="vertical-align:top;">
  <span style="text-align:left">

  Расположение конфигурационного файла службы auditd. По умолчанию `/etc/audit/auditd.conf`

  </span>
</td></tr>

<tr><td style="vertical-align:top;">
  <span style="text-align:left">

  `auditd_rules_bak_name`

  </span>
</td>
<td style="vertical-align:top;">
  <span style="text-align:left">

  Название файла архива для сохранения использовавшихся на источнике правил журналирования.
  По умолчанию `rules_backup.tar`

  </span>
</td></tr>

<tr><td style="vertical-align:top;">
  <span style="text-align:left">

  `auditd_rules_dir`

  </span>
</td>
<td style="vertical-align:top;">
  <span style="text-align:left">

  Каталог с правилами журналирования. По умолчанию `/etc/audit/rules.d`

  </span>
</td></tr>

<tr><td style="vertical-align:top;">
  <span style="text-align:left">

  `auditd_siem_ruleset`

  </span>
</td>
<td style="vertical-align:top;">
  <span style="text-align:left">

  Название создаваемого файла с правилами аудита. По умолчанию `00-siem.rules`

  </span>
</td></tr>

<tr><td style="vertical-align:top;">
  <span style="text-align:left">

  `auditd_write_logs`

  </span>
</td>
<td style="vertical-align:top;">
  <span style="text-align:left">

  Сохранение в файле журналов событий службы auditd: `true` — включить, `false` — отключить (по умолчанию)

  </span>
</td></tr>

<tr><td style="vertical-align:top;" colspan="2">
  <span style="emphasis; font-weight:bold;">
    Параметры службы журналирования
  </span>
</td></tr>

<tr><td style="vertical-align:top;">
  <span style="text-align:left">

  `rsyslog_config`

  </span>
</td><td style="vertical-align:top;">
  <span style="text-align:left">

  Расположение конфигурационного файла службы rsyslog. По умолчанию `/etc/rsyslog.conf`

  </span>
</td></tr>

<tr><td style="vertical-align:top;">
  <span style="text-align:left">

  `rsyslog_options`

  </span>
</td>
<td style="vertical-align:top;">
  <span style="text-align:left">

  Список переопределяемых параметров службы rsyslog в формате YAML

  </span>
</td></tr>

<tr><td style="vertical-align:top;">
  <span style="text-align:left">

  `rsyslog_siem_config`

  </span>
</td>
<td style="vertical-align:top;">
  <span style="text-align:left">

  Расположение создаваемого файла с параметрами работы службы rsyslog. По умолчанию `/etc/rsyslog.d/10-siem.conf`

  </span>
</td></tr>

<tr><td style="vertical-align:top;">
  <span style="text-align:left">

  `syslog_ng_config`

  </span>
</td>
<td style="vertical-align:top;">
  <span style="text-align:left">

  Расположение конфигурационного файла службы syslog-ng. По умолчанию `/etc/syslog-ng/syslog-ng.conf`

  </span>
</td></tr>

<tr><td style="vertical-align:top;">
  <span style="text-align:left">

  `syslog_ng_siem_config`

  </span>
</td>
<td style="vertical-align:top;">
  <span style="text-align:left">

  Расположение создаваемого файла с параметрами работы службы syslog-ng. По умолчанию `/etc/syslog-ng/10-siem.conf`

  </span>
</td></tr>

<tr><td style="vertical-align:top;">
  <span style="text-align:left">

  `syslog_options`

  </span>
</td><td style="vertical-align:top;">
  <span style="text-align:left">

  Название службы журналирования источника. По умолчанию `'rsyslogd|syslog-ng'`

  </span>
</td></tr>

<tr><td style="vertical-align:top;" colspan="2">
  <span style="emphasis; font-weight:bold;">
    Дополнительные параметры
  </span>
</td></tr>

<tr><td style="vertical-align:top;">
  <span style="text-align:left">

  `linux_supported`

  </span>
</td>
<td style="vertical-align:top;">
  <span style="text-align:left">

  Список поддерживаемых ОС семейства Linux в формате YAML

  </span>
</td></tr>

<tr><td style="vertical-align:top;">
  <span style="text-align:left">

  `packages_dirs`

  </span>
</td>
<td style="vertical-align:top;">
  <span style="text-align:left">

  Список каталогов для загрузки обновлений поддерживаемых ОС в формате YAML

  </span>
</td></tr>

<tr><td style="vertical-align:top;">
  <span style="text-align:left">

  `upload_local_packages`

  </span>
</td>
<td style="vertical-align:top;">
  <span style="text-align:left">

  Если недоступен репозиторий ОС, использовать обновления из файлов роли Ansible: `true` — использовать,
  `false` — не использовать (по умолчанию)

  </span>
</td></tr>

</tbody>
</table>

<br/>


## Linux-подобные ОС: настройка MaxPatrol SIEM

```
Если в событиях от источника не указан его часовой пояс или часовой пояс 
отличается от UTC+3, в профиле, используемом задачей на сбор событий, нужно вручную указать часовой пояс. Если в 
событиях источника часовой пояс указан неверно — нужно ввести поправку для часового пояса. Если неверно указано
время — нужно ввести поправку для времени.
```

Для получения событий с источника в MaxPatrol SIEM нужно создать и запустить задачу на сбор событий с профилем SysLog.

► Чтобы создать и запустить задачу на сбор событий с источника:

1. В главном меню в разделе **Сбор данных** выберите пункт **Задачи**.

   Откроется страница **Задачи по сбору данных**.

2. В панели инструментов нажмите кнопку **Создать задачу** и в раскрывшемся меню выберите пункт **Сбор данных**.

   Откроется страница **Создание задачи на сбор данных**.

3. В поле **Название** введите название задачи.

4. В раскрывающемся списке **Профиль** выберите **SysLog**.

5. Если требуется, в раскрывающемся списке **Агент** выберите MP 10 Agent для сбора событий.

   В панели **Расписание** вы можете включить и настроить автоматический запуск задачи по расписанию.

6. Нажмите кнопку **Сохранить и запустить**.


Задача на сбор событий с источника создана и запущена.