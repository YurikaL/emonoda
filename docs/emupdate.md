### Описание

**emupdate** - команда для автоматического обновления раздач прямо с трекеров. Она умеет скачивать новые файлы в тот же каталог, где лежали прошлые раздачи, выводить список обновленных файлов, а так же отправлять об этом оповещения. Действует для трекеров с обновляемыми раздачами (http://rutracker.org, http://nnm-club.me и т.д.).

| Скриншот **emupdate** в консоли |
|---------------------------------|
| [<img src=https://raw.githubusercontent.com/wiki/mdevaev/emonoda/emupdate.png height=150>](https://raw.githubusercontent.com/wiki/mdevaev/emonoda/emupdate.png) |

***

### Принцип работы

В секции `trackers` файла `~/.config/emonoda.yaml` указывается [список трекеров](trackers) и регистрационные данные пользователя на них (для неанонимных сайтов). Конфигурация выглядит примерно так:

```yaml
trackers:
    rutracker.org:
        login: xxx
        passwd: secretxxx

    nnm-club.me:
        login: yyy
        passwd: secretyyy

    rutor.org: {} # Анонимный трекер, указание логина и пароля не требуется.

    tr.anidub.com:
        login: zzz
        passwd: secretzzz
```

При запуске, **emupdate** логинится на трекерах, сканирует торрент-файлы в каталоге, указанном в параметре `core/torrents_dir` и обрабатывает каждый из них следующим образом:

1. Из торрент-файла считывается комментарий, содержащий URL страницы, откуда файл был скачан. По адресу сайта определятся плагин и трекер, который должен обработать этот файл. Например для торрента с комментарием `http://rutracker.org/forum/viewtopic.php?t=4309797` будет выбран плагин `rutracker.org`.

2. Учитывая специфические для трекера методы, **emupdate** посещает считанный URL и смотрит, произошло ли обновление торрента. Так, для рутрекера используется сравнение хешей локального торрента и файла в раздаче, для нонейм-клуба выполняется scrape-запрос, а в случае с анидабом проверяется время модификации локального торрент-файла и время обновления раздачи, с учетом вашего часового пояса на сайте.

3. Если раздача обновилась, **emupdate** скачивает новый торрент-файл взамен старого. При этом старый торрент может быть опционально сохранен в каталоге с бекапами и с временной отметкой, когда он был помещен на хранение.

4. Если вы настроили [интеграцию с клиентом](clients), то **emupdate** удалит из него старую раздачу и добавит обновленную. Никаких дополнительных действий вам не потребуется, все происходит в автоматическом режиме. Ваш клиент скачает файлы в новой раздаче, сохранив их по тому же пути, по которому сохранялись данные в прошлой. Кроме того, если ваш клиент это поддерживает (как, например, **rtorrent**), можно сохранить метки раздач и другие их атрибуты.


***
### Опции

!!! info
    Кроме собственных опций, указанных ниже, **emupdate** поддерживает еще и общий стандартный набор (например, `--help` и `--config`). За подробностями обратитесь на страницу [config](config).

* **`-f, --name-filter <pattern>`**
    * **emupdate** сканирует каталог, указанный в конфигурационном файле параметром `core/torrents_dir`, выбирая из него файлы по шаблону `*.torrent`. С помощью этой опции шаблон можно изменить, например так: `-f 'adventure_time*.torrent`. Это позволяет проверять только интересующие прямо сейчас раздачи. Опция переопределяет значение параметра `emupdate/name_filter`.

* **`-y, --only-trackers <plugin1 plugin2 ...>`**
    * Проверять только торренты для указанного подмножества трекеров от всех настроенных в секции `trackers`.

* **`-x, --exclude-trackers <plugin1 plugin2 ...>`**
    * Включить все трекеры из `trackers`, кроме перечисленных в этой опции.

* **`--noop`**
    * Только проверить наличие обновлений, но не заменять локальные торрент-файлы и не трогать их в клиенте.

* **`--mute`**
    * Даже если настроены оповещения об обновлениях в секции `confetti`, не рассылать их.

* **`--fail-on-captcha`**
    * Некоторые трекеры могут запрашивать в редких случаях капчу при логине. В этом случае **emupdate** выдаст вам ссылку на картинку с капчей и попросит ввести в терминале ее раскодированное значение. Если указана эта опция, то **emupdate** упадет, вместо того, чтобы требовать капчу. Это полезно при запуске программы через крон.


***
### Конфигурационные параметры

Общие параметры и способ настройки описаны на странице [config](config), здесь же приведены специфические параметры программы.

* **`emupdate/name_filter='*.torrent'`**
    * Шаблон, которому должны соответствовать файлы из каталога `core/torrents_dir`. Переопределяется опцией `--name-filter`.

* **`emupdate/backup_dir=""`**
    * Каталог для бекапов старых торрент-файлов. В конце имени сохраненного файла дописывается время его помещения в бекап (либо что-то, что вы можете настроить параметром `emupdate/backup_suffix`.

* **`emupdate/backup_suffix=.%Y.%m.%d-%H:%M:%S.bak`**
    * К имени забекапленного торрента дописывается этот суффикс. Например, если торрент назывался `archlinux.torrent`, то имя бекапа с суффиксом по умолчанию будет выглядеть так: `archlinux.torrent.2015.10.21-22:46:26.bak`. Список доступных форматтеров смотрите [тут](http://docs.python.org/3/library/datetime.html#strftime-strptime-behavior).

* **`emupdate/save_customs=[]`**
    * У [некоторых клиентов](clients) есть особый набор полей данных, привязанных к торрентам (например, метки файлов). Эти поля доступны через API для чтения и запись. При обновлении торрента можно указать, какие из меток должны быть сохранены. Скажем, если вы используете **ruTorrent**, метки файлов будут храниться внутри **rtorrent**, доступные через [XMLRPC](rTorrent-XMLRPC-Reference) через методы `d.set_custom1/d.get_custom1`, и в **emonoda** им будет соответствовать поле `c1`. Таким образом, для сохранения меток торрентов при их обновлении вам потребуется указать параметр следующим образом (в виде списка):

  ```yaml
  emupdate:
      save_customs: [c1]
  ```

  или:

  ```yaml
  emupdate:
      save_customs:
          - c1
  ```

* **`emupdate/set_customs={}`**
    * Работает почти так же, как и `emupdate/save_customs`, но вместо сохранения назначает обновленным торрентам новые значения кастомных полей. Поддерживает те же [форматтеры времени](http://docs.python.org/3/library/datetime.html#strftime-strptime-behavior), что и `emupdate/backup_suffix`. Например, следующая конфигурация заставит **emupdate** при использовании **ruTorrent** назначить метку `new` на все новые раздачи:

  ```yaml
  emupdate:
      set_customs:
          c1: new
  ```

* **`emupdate/show_diff=true`**
    * Если торрент был обновлен, показать, какие файлы изменились по сравнению с предыдущей его версией. Результат аналогичен выводу команды [emdiff](emdiff). Удобно для слежения за новыми сериями в сериалах.

* **`emupdate/show_passed=false`**
    * Этот параметр определяет, нужно ли показывать в логе записи о не обновленных торрентах. В большинстве случаев это не нужно.

* **`emupdate/show_unknown=false`**
    * Этот параметр определяет, нужно ли показывать в логе записи о торрентах, для которых не нашлось подходящего трекерного плагина. В большинстве случаев это не нужно.


***
### Обработка ошибок

Во время работы программа выводит на терминал построчный лог прогресса. Иногда вместо названия трекера могут появляться вот такие сообщения:

```
[!] 1/2 UNKNOWN foo.torrent -- http://example.com
[!] 2/2 NOT_IN_CLIENT bar.torrent -- http://example.com
```
Это означает, что у **emupdate** не получилось проверить обновления торрента.

Причины могут быть разными:

| Ошибка | Описание |
|:------:|----------|
| `UNKNOWN` | Возникает, когда не удалось распознать трекер, с которого был скачан торрент (в комментарии к нему не написан урл, либо поддержка трекера не реализована). Обычно пропускается и не показывается в логе, но можно включить отображение таких торрентов с помощью параметра `emupdate/show_passed=true`.  |
| `NOT_IN_CLIENT` | Если торрент-файл не зарегистрирован в клиенте. |
| `INVALID_TORRENT` | Локальный торрент-файл побился и содержит мусор вместо валидного описания раздачи. |


***
### Оповещения

Программа умеет отсылать оповещения при обновлении торрентов. Оповещения настраиваются почти так же, как и трекеры - перечислением плагинов с настройками в секции `confetti`. Они полезны в случае, когда вы хотите поставить **emupdate** в крон и получать новости, например, на почту. Следует отметить, что собственные оповещения программы - не замена логам. Информация о сбоях присылаться не будет, и при использовании в кроне настоятельно рекомендуется перенаправлять `stdout` и `stderr` от **emupdate** в файл. О том, как настроить оповещения об обновленных раздачах, читайте на странице [confetti](confetti).

| Пример письма |
|---------------|
| [<img src=https://raw.githubusercontent.com/wiki/mdevaev/emonoda/emupdate_email.png height=150>](https://raw.githubusercontent.com/wiki/mdevaev/emonoda/emupdate_email.png) |