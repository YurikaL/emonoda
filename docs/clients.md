!!! info
    **Emonoda** должна стоять на одной машине с клиентом, поскольку ее программам требуется прямой доступ к файлам. О базовых принципах настройки системы читайте на странице, [посвящённой конфигурации](config). Клиентские плагины имеют множество параметров, но в обычной ситуации достаточно значений по умолчанию. Их вы можете посмотреть с помощью `emfile -m`.


***
### rTorrent

Включается параметром `core/client=rtorrent`. Соединение с клиентом выполняется по урлу, задаваемому параметром `client/url`, по умолчанию - `http://localhost/RPC2`.

Для использования этого клиента вам потребуется веб-сервер с настроенным XMLRPC-шлюзом. Если вы используете [ruTorrent](https://github.com/Novik/ruTorrent), то все необходимые настройки у вас уже должны быть выполнены. В противном случае воспользуйтесь параграфом "Настройка веб-сервера" из [этого](https://wiki.archlinux.org/index.php/RuTorrent_(%D0%A0%D1%83%D1%81%D1%81%D0%BA%D0%B8%D0%B9)) руководства.

Поддерживается полный набор функций, включая кастомные поля данных - атрибуты, которые позволяет сохранять **rTorrent** для каждого торрент-файла. Например, **ruTorrent** в поле `c1` хранит тег раздачи. Доступны `c1`, `c2`, `c3`, `c4` и `c5` - по аналогии с методами `d.set_custom1/d.get_custom1` [API rTorrent](rTorrent-XMLRPC-Reference).


***
### Transmission
Включается параметром `core/client=transmission`. Соединение с клиентом выполняется по урлу, задаваемому параметром `client/url`, по умолчанию - `http://localhost:9091/transmission/rpc`.

Для работы нужен питоновый модуль [transmissionrpc](https://bitbucket.org/blueluna/transmissionrpc) версии 0.11 или выше. Нет поддержки сохранения кастомных полей (по причине их [отсутствия](https://trac.transmissionbt.com/ticket/2175) в самом Transmission).

Логин и пароль задаются параметрами `client/user` и `client/passwd` соответственно.


***
### KTorrent

Включается параметром `core/client=ktorrent`. Соединение выполняется с помощью DBus.

Поддержка этого клиента сильно ограничена из-за его куцего API. Кроме того, для использования вместе с **Emonoda** вам потребуется убрать в настройках **KTorrent** путь по умолчанию для сохранения скачанных данных ("Папка по умолчанию для загрузки"), иначе [emupdate](emupdate) не сможет корректно обновлять ваши раздачи.

Для работы нужен питоновый модуль **dbus**. Нет поддержки кастомных полей.