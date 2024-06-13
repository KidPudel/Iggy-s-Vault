```sh
pg_dump -U postgres -h localhost -p 5432 -W -F t chinesebee_db > local_backup          
```

```sh
pg_restore -U postgres -h monorail.proxy.rlwy.net -p 47948 -W -F t -d railway local_backup
```
