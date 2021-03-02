# SEQUELIZE

The possible isolations levels to use when starting a transaction:

```javascript
const { Transaction } = require("sequelize");

// The following are valid isolation levels:
Transaction.ISOLATION_LEVELS.READ_UNCOMMITTED; // "READ UNCOMMITTED"
Transaction.ISOLATION_LEVELS.READ_COMMITTED; // "READ COMMITTED"
Transaction.ISOLATION_LEVELS.REPEATABLE_READ; // "REPEATABLE READ"
Transaction.ISOLATION_LEVELS.SERIALIZABLE; // "SERIALIZABLE"
```

By default, sequelize uses the isolation level of the <span style="color:RED">_database_</span>.

If you want to use a different isolation level, pass in the desired level as the first argument:

```javascript
const { Transaction } = require("sequelize");

await sequelize.transaction(
  {
    isolationLevel: Transaction.ISOLATION_LEVELS.SERIALIZABLE,
  },
  async (t) => {
    // Your code
  }
);
```

You can also overwrite the isolationLevel setting globally with an option in the Sequelize constructor:

```javascript
const { Sequelize, Transaction } = require("sequelize");

const sequelize = new Sequelize("sqlite::memory:", {
  isolationLevel: Transaction.ISOLATION_LEVELS.SERIALIZABLE,
});
```

# Transaction sandbox for psql

>

## Attach to psql in docker container

```bash
docker exec -it local_db_container sh -c "psql -U slaveAUser"
```

## Проверка текущего состояния записи

```sql
SELECT email FROM kusers WHERE name='Denzel';
```

---

## Создание записи в таблице

```sql
INSERT INTO kusers ("name") VALUES ('SER1');
```

## READ_COMMIT

### СУТЬ: видит внутри транзакцию любые <span style="color:Blue">COMMIT</span> изменения другой транзакции

### ДРУГИЕ ТРАНЗАКЦИИ:

- READ: видет все <span style="color:Blue">COMMIT</span> изменения другой транзакции.
- UPDATE: В этом режиме строки, к которым обращается транзакция для чтения или записи, блокируются. canceling statement due to user request
- CREATE: В этом режиме строки, к которым обращается транзакция для чтения или записи, блокируются. canceling statement due to user request
- DELETE: В этом режиме строки, к которым обращается транзакция для чтения или записи, блокируются. canceling statement due to user request

```sql
BEGIN;

UPDATE kusers SET email='tr1' WHERE name='Denzel';

COMMIT;
```

### ЗАДАЧА:

con1: начать транзакцию
con2: удалить любую запись

что случиться?

### Водичка

> _Несогласованность_ данных. Предположим, в контексте первой транзакции исполняется запрос на определение числа записей в таблице. По завершении этого запроса во второй транзакции проводится удаление и/или добавление записей в таблицу. Если теперь первая транзакция заново выполнит запрос на количество записей в таблице, результат будет отличаться от первоначально полученного значения.

---

## REPEATABLE_READ

### СУТЬ: видет только те записи и их состояние которое было прочитано _до начала транзакци_

### ДРУГИЕ ТРАНЗАКЦИИ:

- READ: видет только те записи и их состояние которое было прочитано до начала транзакци.
- UPDATE: В этом режиме строки, к которым обращается транзакция для чтения или записи, блокируются. canceling statement due to user request
- CREATE: В этом режиме строки, к которым обращается транзакция для чтения или записи, блокируются. canceling statement due to user request
- DELETE: В этом режиме строки, к которым обращается транзакция для чтения или записи, блокируются. canceling statement due to user request

```sql
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

UPDATE kusers SET email='tr1' WHERE name='Denzel';

INSERT INTO kusers ("name") VALUES ('SER1');

COMMIT;
```

### ЗАДАЧА:

con1: начать транзакцию
con2: удалить любую запись

что случиться?

### Водичка

> _Строки-призраки_ - Транзакция может заблокировать все записи, с которыми идет работа, но другая транзакция в это время может добавить строки в таблицу. Поэтому, когда между двумя чтениями в одной транзакции другая добавляет строки, возникают так называемые "строки-призраки", так как она внезапно появляется в процессе работы одной транзакции.

---

## SERIALIZABLE

### СУТЬ: блокировать таблицу для изменений

### ДРУГИЕ ТРАНЗАКЦИИ:

- READ: видет только те записи их состояние которое было прочитано до начала транзакци
- UPDATE: если попытаться провести, выдаст ошибку ERROR: current transaction is aborted, commands ignored until end of transaction block
- CREATE: если попытаться провести, выдаст ошибку ERROR: current transaction is aborted, commands ignored until end of transaction block
- DELETE: если попытаться провести, выдаст ошибку ERROR: current transaction is aborted, commands ignored until end of transaction block

```sql
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;

UPDATE kusers SET email='tr1' WHERE name='Denzel';

COMMIT;
```

# Links

- [Детальное описание](http://kharchuk.ru/home/9-%D0%9F%D1%80%D0%BE%D1%87%D0%B5%D0%B5/53-mysql-transactions)

- [Документация](https://postgrespro.ru/docs/postgrespro/10/tutorial-transactions)

- [Sequelize](https://sequelize.org/master/manual/transactions.html)
- [Youtube](https://www.youtube.com/watch?v=4EajrPgJAk0&ab_channel=TECHSCHOOL)
