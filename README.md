Есть только такой пруф того, что я сделал дз до дд, а потом забыл его сдать. На случай если вы меня решите помиловать(
<img width="1076" height="198" alt="image" src="https://github.com/user-attachments/assets/ed2a9745-d12a-4fd5-8438-b3c25f63529c" />
А вот время коммита, c которым оно совпадает
<img width="681" height="71" alt="image" src="https://github.com/user-attachments/assets/6f46ad92-c826-48d5-8187-bfa2e93dda63" />

## 4 балла — один контейнер
Команды сборки и запуска:
```bash
docker build -t counter-app:1 .
docker run -d --name counter -p 127.0.0.1:8080:8000 -e GREETING='Привет из Docker!' counter-app:1
```
Проверка работы:
```bash
curl -fsS http://127.0.0.1:8080/
curl -fsS -X POST http://127.0.0.1:8080/increment
curl -fsS -X POST http://127.0.0.1:8080/increment
curl -fsS http://127.0.0.1:8080/
```
```
{"greeting": "Привет из Docker!", "count": 0, "hostname": "3dc877887d1c", "storage": "sqlite"}% 

{"greeting": "Привет из Docker!", "count": 1, "hostname": "3dc877887d1c", "storage": "sqlite"}%   

{"greeting": "Привет из Docker!", "count": 2, "hostname": "3dc877887d1c", "storage": "sqlite"}%   

{"greeting": "Привет из Docker!", "count": 2, "hostname": "3dc877887d1c", "storage": "sqlite"}%  
```
Логи:
```bash
docker logs counter
```
```
2026-11-24 13:28:08,738 listening port=8000 storage=sqlite
2026-11-24 13:28:27,698 http 172.17.0.1 "GET / HTTP/1.1" 200 -
2026-11-24 13:28:32,926 http 172.17.0.1 "POST /increment HTTP/1.1" 200 -
2026-11-24 13:28:37,173 http 172.17.0.1 "POST /increment HTTP/1.1" 200 -
2026-11-24 13:28:40,719 http 172.17.0.1 "GET / HTTP/1.1" 200 -
2026-11-24 13:28:44,951 http 172.17.0.1 "GET /readyz HTTP/1.1" 200 -
```
---
## 6 баллов — Compose и данные
Проверка сохранения счётчика после удаления и повторного создания контейнера:
```bash
docker compose up -d --build --wait
for i in 1 2 3; do curl -fsS -X POST http://127.0.0.1:8080/increment; echo; done
curl -fsS http://127.0.0.1:8080/
docker compose down
docker volume ls
docker compose up -d --wait
curl -fsS http://127.0.0.1:8080/
```
```
[+] up 4/4
 ✔ Image counter-hw-app                Built                                                2.1s
 ✔ Volume counter-hw_counter-data      Created                                              0.0s
 ✔ Network counter-hw_default          Created                                              0.0s
 ✔ Container counter-hw-app-1          Healthy                                              0.6s

{"greeting": "Hello from Compose!", "count": 1, "hostname": "1627a183823d", "storage": "sqlite"}
{"greeting": "Hello from Compose!", "count": 2, "hostname": "1627a183823d", "storage": "sqlite"}
{"greeting": "Hello from Compose!", "count": 3, "hostname": "1627a183823d", "storage": "sqlite"}
{"greeting": "Hello from Compose!", "count": 3, "hostname": "1627a183823d", "storage": "sqlite"}%   

[+] down 2/2
 ✔ Container counter-hw-app-1          Removed                                              3.1s
 ✔ Network counter-hw_default          Removed                                              0.1s

DRIVER    VOLUME NAME
local     counter-hw_counter-data
local     counter-hw_redis-data

 ✔ Network counter-hw_default          Created                                              0.0s
 ✔ Container counter-hw-app-1          Healthy                                              0.6s

 {"greeting": "Hello from Compose!", "count": 3, "hostname": "c697623ed8a0", "storage": "sqlite"}%        
```
Проверка настраиваемости порта и приветствия:
```bash
HOST_PORT=9090 GREETING='Привет!' docker compose up -d --wait
curl -fsS http://127.0.0.1:9090/
```
```
 ✔ Container counter-hw-app-1          Healthy                                              3.7s

 {"greeting": "Привет!", "count": 3, "hostname": "ea01759adb2e", "storage": "sqlite"}% 
```
---
## 8 баллов — отдельное хранилище
Состояние сервисов:
```bash
docker compose ps
```
```
NAME                 IMAGE              COMMAND                  SERVICE   CREATED          STATUS                    PORTS
counter-hw-app-1     counter-hw-app     "python app.py"          app       15 seconds ago   Up 9 seconds (healthy)    127.0.0.1:8080->8000/tcp
counter-hw-redis-1   redis:7.4-alpine   "docker-entrypoint.s…"   redis     15 seconds ago   Up 15 seconds (healthy)   6379/tcp
```
### Проверка 1: данные после down и up
```
{"greeting": "Hello from Compose!", "count": 23, "hostname": "ddf0c93c5ee1", "storage": "redis"}%   
```
### Проверка 2: Redis остановлен
```bash
docker compose stop redis
curl -s -o /dev/null -w "healthz: %{http_code}\n" http://127.0.0.1:8080/healthz
curl -s -o /dev/null -w "readyz: %{http_code}\n" http://127.0.0.1:8080/readyz
curl -s -o /dev/null -w "POST increment: %{http_code}\n" -X POST http://127.0.0.1:8080/increment
```
```
[+] stop 1/1
 ✔ Container counter-hw-redis-1        Stopped                                              0.2s

healthz: 200

readyz: 503

POST increment: 503
```
### Проверка 3: Redis вернулся
```bash
docker compose start redis
curl -fsS http://127.0.0.1:8080/
curl -fsS -X POST http://127.0.0.1:8080/increment
```
```
[+] start 1/1
 ✔ Container counter-hw-redis-1        Started                                              0.1s

 {"greeting": "Hello from Compose!", "count": 23, "hostname": "ddf0c93c5ee1", "storage": "redis"}% 

 {"greeting": "Hello from Compose!", "count": 24, "hostname": "ddf0c93c5ee1", "storage": "redis"}%  
```
---
