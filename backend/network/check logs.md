If we want to check logs of the systemd (incase of -u service unit)
```
journalctl -u OTC_Test5 -f
```

if we want to see logs of container
1. check the info (hash)
```
docker ps | grep <name>
```
2. check last 100 logs
```
docker logs 1dc0 -n 100
```
or relative
```
docker logs 4c07432c8b25 --until "9m"
```
also could be since



-n <num>: number of logs
-f: follow
--since