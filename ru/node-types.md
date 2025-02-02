# Типы нод

Нода VIZ — сердце блокчейна, программное обеспечение, которое обрабатывает блок за блоком, исполняет все операции и хранит состояние системы. Владелец настраивает ноду, выбирает используемые плагины. От включенных плагинов и их настроек зависит то, какие возможности может предоставлять нода.

***

## Witness node (нода делегата)

За формирование блоков отвечают делегаты, поэтому важнейшими нодами являются как раз ноды делегатов. Они обмениваются данными с другими нодами, собирают транзакции. И когда подходит очередь делегата (владельца сервера) сформировать блок — происходит подпись собранного блока и его трансляция другим узлам. Блок должен быть подписан и доставлен за 3 секунды, выделенные на это делегату. Пропуск блока приводит к задержке выполнения транзакций и штрафу делегата на какое-то время (штраф накладывается на суммарный вес голосов, отданных за делегата другими пользователями). Это позволяет системе временно понижать в очереди делегатов тех, у кого случились какие-то неполадки на сервере или дата-центре, защищая таким образом надежность сети.

Зачастую делегаты держат две ноды, основную и запасную (резерв). Часто они находятся в разных дата-центрах и не зависят друг от друга. Если с основной нодой происходит неполадка, то делегат меняет ключ подписи блоков на резервный и формированием блоков будет заниматься запасная нода.

Ключевые плагины: `chain p2p json_rpc webserver witness network_broadcast_api database_api witness_api`

## Seed node (сид-нода)

Основа для функционирования любой блокчейн системы — peer-to-peer (p2p) соединение и обмен данными. Сид-ноды отличаются тем, что выполняют важную роль — принимают и раздают блоки, разгружая таким образом пропускную способность всей сети и снижая отклик для территориально близких подключений. Нет экономической выгоды держать сид-ноду, так как сервер стоит денег, но не приносит своему владельцу ничего. Поэтому большинство сид-нод запускают делегаты (witnesses), если они могут позволить себе это финансово.

Ключевые плагины: `chain p2p`

## API node

Часть плагинов занимаются предоставляем API для разработчиков и их пользователей. Такие ноды часто называют полными, если у них включены все доступные плагины, и они хранят историю с первого блока. Так как плагины дают доступ не только к состоянию системы, но и формируют свои структуры данных, у них повышенные требования к ресурсам сервера (особенно к оперативной памяти, так как нода хранит [базу данных ChainBase](https://github.com/VIZ-Blockchain/chainbase/tree/c8c527e56740857e29656eee4ba9f88c63063a1b) в RAM). Именно через API ноды происходят запросы на актуальную информацию, статус аккаунтов, историю операций или заявки в комитете.

Примеры таких плагинов:
 - **network_broadcast_api** — отправка транзакции в сеть;
 - **database_api** — основной плагин, предоставляющий доступ к состоянию системы, получению данных об аккаунтах, получению информации о блоке, получению параметров сети;
 - **custom_protocol_api** — плагин для записи счетчика и высоты блока для последних custom операций для аккаунта (количество хранимых идентификаторов кастомных операций для аккаунта задается параметром custom-protocol-store-size в конфигурационном файле);
 - **account_history** — получение списка операций, связанных с аккаунтом;
 - **committee_api** — получение списка заявок по статусу, получение информации о заявке, получение списка голосов по заявке;
 - **invite_api** — получение списка инвайтов по статусу, запрос информации о инвайте по идентификатору или ключу;
 - **operation_history** — получение информации об операциях в блоке;
 - **paid_subscription_api** — получение информации о платных подписках, заключенных соглашениях между аккаунтами;
 - **witness_api** — получение списка делегатов, очереди делегатов, информации о конкретном делегате и его голосуемых параметрах сети.

API ноды могут быть как приватными (когда в настройках указаны параметры доступа по логину и паролю), так и публичными (когда обращение к API доступно всем и публично известен адрес ноды). Часто публичные API ноды называют просто публичными нодами. Подробнее читайте в разделе [Плагины и их API](plugins-api.md).