# Состояние (стэйт) системы

Состояние системы — минимум данных, необходимый для функционирования ноды. Каркас Graphene, на котором построен VIZ, исполняет операции из транзакций в каждом блоке, который соответствует консенсусу (очереди делегатов, соответствие подписей). Каждый блок происходит обработка данных, отложенных действий, что приводит к итерационной сущности состояния системы. Часть данных хранятся там постоянно, что накладывает определенные требования на серверное оборудование, на котором запущена нода.

> В разделе [Объекты и структуры в блокчейне](object-structures.md) описана большая часть объектов, которые составляют состояние системы.

***

## Dynamic global property object (dgpo)

Данный объект хранит основные свойства сети, содержит информацию о токенах в обращении и другую важную информацию. В исходном коде он часто представлен в виде переменной dgp, нода модифицирует его состояние с каждой иттерацией, поэтому dynamic global property можно с уверенностью считать самой важной частью состояния системы. Рассмотрим его публичные свойства, доступные через метод get_dynamic_global_properties при обращении к плагину database_api:

 - **current_witness** (пример значения: "solox") — делегат актуального блока;
 - **head_block_number** (пример значения: 10829669) — номер актуального блока;
 - **head_block_id** (пример значения: `00a53f658226b2a6f3f75fc8185d884d029f50bf`) — идентификатор (он же хэш) актуального блока;
 - **time** (пример значения: "2019-10-11T08:49:21") — время генерации актуального блока;

 - **last_irreversible_block_num** (пример значения: 10829651) — номер последнего необратимого блока;
 - **genesis_time** (пример значения: "2018-09-29T10:23:24") — время генерации первого блока сети;

 - **current_supply** (пример значения: "55159321.957 VIZ") — общее количество токенов VIZ в системе;
 - **total_vesting_fund** (пример значения: "27924855.425 VIZ") — количество токенов VIZ переведенных в долю сети (SHARES);
 - **total_vesting_shares** (пример значения: "27924847.935329 SHARES") — общая количественная мера доли сети в SHARES;
 - **committee_fund** (пример значения: "1423813.837 VIZ") — баланс фонда комитета;
 - **committee_requests** (пример значения: 39) — общее количество заявок в комитет;
 - **total_reward_fund** (пример значения: "28216.906 VIZ") — баланс фонда наград;
 - **total_reward_shares** (пример значения: "6019776735774") — количественная мера конкуренции за фонд наград;

 - **current_aslot** (пример значения: 10855719) — текущий номер слота делегата на подпись (содержит в себе нумерацию слота от старта в сети, включая пропущенные делегатами блоки);
 - **recent_slots_filled** (пример значения: `340282366920938463463374607431768211455`) — используется для вычисления процента делегатов участвующих в подписи блоков;
 - **participation_count** (пример значения: 128) — необходимо разделить на 128, чтобы получить процент делегатов участвующих  в подписи блоков;

 - **maximum_block_size** (пример значения: 65536) — максимальный размер блока в байтах (голосуемый параметр сети);
 - **average_block_size** (пример значения: 114) — средний размер блока, рассчитывается по формуле `average_block_size = (99 * average_block_size + new_block_size) / 100`, используется для обновления current_reserve_ratio для поддержания около 50% или меньше в пропускной способности сети;
 - **max_virtual_bandwidth** (пример значения: `5986734968066277376`) — максимальная пропускная способность сети рассчитывается по формуле `maximum_block_size * CHAIN_BANDWIDTH_AVERAGE_WINDOW_SECONDS / CHAIN_BLOCK_INTERVAL`, максимальная виртуальная пропускная способность сети по формуле `max_bandwidth * current_reserve_ratio`
 - **current_reserve_ratio** (пример значения: 20000) — Раз в 20 блоков (1 минута) происходит проверка `average_block_size <= 25% maximum_block_size`. Если оно выполняется, то данное значение увеличивается на 1 (линейно, каждый блок), но не более CHAIN_MAX_RESERVE_RATIO (20000). Если условие не выполнено, то current_reserve_ratio делится пополам, что должно сразу снизить нагрузку на сеть, защищая ее от участников, использующих объемные транзакции. Другими словами, двукратное уменьшение резервного соотношения не уменьшит вдвое использование сети, но ограничит пользователей, которые уже попытаются превысить более 50% от их пропускной способности. Когда резервное соотношение падает вдвое от максимального значения (10000 вместо 20000), восстановление общей виртуальной пропускной способности займет около 7 суток.

 - **bandwidth_reserve_candidates** (пример значения: 1) — количество кандидатов на резерв пропускной способности (плюс 1 кандидат по умолчанию);
 - **inflation_calc_block_num** (пример значения: 10315901) — номер блока последнего расчета распределения инфляции;
 - **inflation_witness_percent** (пример значения: 2000) — процент от эмиссии, получаемый делегатами за подпись блоков;
 - **inflation_ratio** (пример значения: 5000) — процент соотношения от эмиссии, направляемый в фонд комитета против фонда наград;
 - *vote_regeneration_per_day* (пример значения: 1) — устаревшее свойство.

## Уникальность транзакций и TaPoS (Transactions as Proof of Stake)

Нода проверяет все входящие транзакции на уникальность в пуле транзакций. После того, как наступает expiration (ограничено в настройках константой CHAIN_MAX_TIME_UNTIL_EXPIRATION в один час), транзакция удаляется из пула.

Все транзакции в VIZ должны [соответствовать концепции TaPoS](https://github.com/super3/invictus.io/blob/master/assets/pdf/TransactionsAsProofOfStake10.pdf), то есть, ссылаться на один из прошлых блоков (ref_block_num в 2 байтовом представлении (бинарное «и» десятичного представления номера блока с hex `ffff`) и ref_block_prefix, состоящий из десятичного представления 5, 6, 7, 8 байтов от бинарного состояния хэша в обратном порядке), что позволяет инициатору транзакции опираться на актуальное для него состояние системы, не беспокоясь о необратимом блоке. В случае, если он опирался на состояние системы в случайном минорном форке, то транзакция не попадет в основную цепочку. Таким образом, участники сети могут контролировать исполнение очереди транзакций и строить взаимодействие, не дожидаясь необратимости блока. Это, в свою очередь, накладывает ограничение на финальный учет подобных действий, поэтому большинство важных транзакций должны находиться уже в необратимом состоянии для проверяющей стороны.

Нода выделяет пространство block_summary_object с размерностью в 2 байта (чтобы номер блока, прошедший через операцию бинарного «и» с hex `ffff`, умещался в диапазоне от 0 до 65536) и, принимая новые блоки, перезаписывает по кругу идентификаторы (хэши) в этом пространстве (и индексе block_summary_index). 65537 блоков охватывают временной промежуток 196611 секунд (примерно 2.27 суток). Соответственно, новые транзакции могут ссылаться только на блоки из этого пространства, чтобы нода могла сверить соответствие идентификатора блока (из ref_block_num) с контрольной суммой из ref_block_prefix.

## Переносимое состояние системы

Если в первом поколении блокчейн-систем, основанных на Proof of Work, требовалось хранить в состоянии системы все идентификаторы блоков и транзакций, то с ростом количества данных многие разработчики начали искать способ снизить издержки на объем хранимых данных. И основной объем данных хранится как раз в блоках и содержащихся в них транзакциях. В современных DLT уже решена эта проблема за счет согласования необратимого состояния и переносимого состояния системы. Часть блокчейн-систем только начинают двигаться в этом направлении. Например, в Steem предложено решение в виде [Platform Independent State Files – PISF](https://steemit.com/steem/@steemitblog/blockchain-update-platform-independent-state-files). Новые блокчейн-системы (и часть старых первопроходцев, например, нода к XRP Ledger — rippled) уже созданы с учетом переносимого состояния системы, их архитектура позволяет запросить у доверенных нод актуальное состояние системы, пропуская длительную синхронизацию, скачивание всей истории блокчейна и самостоятельную обработку всех транзакций.

VIZ не вносил значительных изменений в архитектуру состояния системы Graphene, поэтому в индексе block_summary_index, состоящем из структур block_summary_object, хранится вся информация о блоках (а именно block_id_type конкретного блока, который уже содержит всю информацию). Это затрудняет создание переносимого состояния системы, так как объем данных для такого состояния будет значительным. Единственная возможность модернизировать это — перейти к консенсусу доверенных нод.