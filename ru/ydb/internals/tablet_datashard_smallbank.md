#### Описание 

В статье "A Critique of ANSI SQL Isolation Levels" описываются 8 типов аномалий, которые могут возникать в случае ошибок реализации изоляции одновременно выполняющихся транзакций.

В работе "Serializable Isolation for Snapshot Databases" приведен тест SmallBank на аномалию write skew - единственную из перечня аномалий, проявляющуюся при snapshot-изоляции (например, на базе MVCC). У нас реализована несколько модифицированая версия этого теста в 3 вариантах: на чистом MiniKQL, на MiniKQL с локами и на SQL. Вариант SQL внутри БД транслируется в (более сложный) MiniKQL с локами. Тест покрывает только точечные локи.

#### Суть теста 

У пользователя (идентифицируемого по UserId) есть два счета в банке: расчетный (Checking) и накопительный (Saving). Банк поддерживает сумму (Checking + Saving) неотрицательной, Saving неотрицательным, и предоставляет три операции по счетам:

- уменьшить Checking (выписать чек)
- увеличить Checking (пополнить расчетный счет)
- изменить Saving (положить или снять деньги с накопительного счета)

Checking может быть отрицательным при условии, что (Checking + Saving) >= 0.

Тест заключается в том, чтобы гарантировать выполнение несовместимых транзакций, сохранив constraint (Checking + Saving) >= 0. Для каждого пользователя одновременно приходит две транзакции: "уменьшить Checking" и "уменьшить Saving", такие что, будь они выполнены вместе, баланс стал бы отрицательным. Выполниться из них должна только одна. Вторая обязана откатиться.

#### Пример использования 

``` bash
Usage: ./small_bank [OPTIONS]

Options:
  {-V|--svnrevision} print svn version
  {-?|--help}        print usage
  {-t|--test} VAL    bank | phantoms (default: "bank")
  {-m|--mode} VAL    sql | mkql | lmkql (default: "sql")
  {-s|--server} VAL  server
  {-p|--port} VAL    port (default: 2134)
  {-r|--root} VAL    root directory name (default: "Root")
  {-u|--users} VAL   number of users (default: 1)
  {-a|--aparts} VAL  number of Checking table's parts (default: 64)
  {-b|--bparts} VAL  number of Saving table's parts (default: 64)
  {-j|--threads} VAL run in N threads (default: 0)
  {-N|--no-init}     don't init schema (default: 0)
  {-D|--duration} VAL time to run (seconds) (default: 0)
```
Запустить на в тестовом энвайронминте (не требует сервера) 8 пар конкурирующих транзакций в 1 поток на MiniKQL c локами
``` bash
./small_bank -m lmkql -j 1 -u 8
```
Запустить 1000 пар конкурирующих транзакций (по 2 для каждого UserId), на localhost-е, на дефолтном порту, в 8 потоков на MiniKQL без локов
``` bash
./small_bank -m mkql -j 8 -u 1000 -s localhost
```
#### Детали реализации 

Заданы две таблички:

- Checking {UserId : Uint64, Balance : Int64}
- Saving {UserId : Uint64, Balance : Int64}

В цикле в многопоточную очередь добавляются пары конкурирующих транзакций. По окончании каждая транзакция читает и валидирует баланс своего UserId. Если он не сходится - выдается ошибка. UserId в тесте геренируются псевдослучайно, чтобы обеспечить размазывание нагрузки по шардам.

Ошибками считаются следующие ситуации:

- Переход суммы балансов (либо баланса Saving) в отрицательную зону
- Невыполнение ни одной из конкурирующих транзакций
- Несовпадение суммы балансов, посчитанной в транзакции и в клиентском коде

Запустить на в тестовом энвайронминте (не требует сервера) 8 пар конкурирующих транзакций в 1 поток на MiniKQL c локами
``` bash
./small_bank -m lmkql -j 1 -u 8
```
Запустить 1000 пар конкурирующих транзакций (по 2 для каждого UserId), на localhost-е, на дефолтном порту, в 8 потоков на MiniKQL без локов
``` bash
./small_bank -m mkql -j 8 -u 1000 -s localhost
```
#### Детали реализации 

Заданы две таблички:

- Checking {UserId : Uint64, Balance : Int64}
- Saving {UserId : Uint64, Balance : Int64}

В цикле в многопоточную очередь добавляются пары конкурирующих транзакций. По окончании каждая транзакция читает и валидирует баланс своего UserId. Если он не сходится - выдается ошибка. UserId в тесте геренируются псевдослучайно, чтобы обеспечить размазывание нагрузки по шардам.

Ошибками считаются следующие ситуации:

- Переход суммы балансов (либо баланса Saving) в отрицательную зону
- Невыполнение ни одной из конкурирующих транзакций
- Несовпадение суммы балансов, посчитанной в транзакции и в клиентском коде