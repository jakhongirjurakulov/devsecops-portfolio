### 3. Установлен Docker и выполнена базовая проверка

Docker установлен на Ubuntu VM, выполнен smoke test
Результат: Docker работает корректно, образы скачиваются и контейнеры запускаются.

### 4. Установлен Portainer (UI для Docker)

Развёрнут Portainer для управления контейнерами через веб-интерфейс (UI), доступ по адресу:
https://localhost:94**
Portainer использовался как удобный интерфейс для просмотра контейнеров, логов и состояния.

### 5. Собраны защищённые Docker-образы

Собраны образы:
* sqb/cards-service:secure
* sqb/payments-service:secure
    
Сборка выполнялась через:
* docker build -t sqb/cards-service:secure ~/devsecops-portfolio/**
* docker build -t sqb/payments-service:secure ~/devsecops-portfolio/**

Ключевые принципы в образах:

* создание отдельного пользователя внутри контейнера  
* запуск процесса не от root (USER) 
* запуск приложения через app.sh

### 6. Запуск containers в "secure runtime" режиме

#### Cards (secure)
Контейнер запущен с параметрами безопасности:
* \--read-only (immutable filesystem)
* \-v ...:ro (конфиг примонтирован read-only)
* \--user : (контейнер работает не от root, UID согласован с хостом)

Пример запуска (логика):
sudo docker run -d \    --name cards-secure \    --read-only \    -v /opt/sqb/cards/config.env:/app/config.env:ro \    --user 1001:1001 \    sqb/cards-service:secure

#### Payments (secure)
Аналогично для payments (UID берётся из id payments\_srv):
sudo docker run -d \    --name payments-secure \    --read-only \    -v /opt/sqb/payments/config.env:/app/config.env:ro \    --user 1002:1002 \    sqb/payments-service:secure

### 7. Валидация: контейнеры реально работают
Проверка статуса контейнеров: docker ps
Результат:
* cards-secure — Running
* payments-secure — Running

### 8. Валидация: контейнер использует реальный конфиг из Linux baseline
Проверка, что конфиг примонтирован с хоста и read-only:
docker inspect cards-secure | grep config.env -A 5  docker inspect payments-secure | grep config.env -A 5

Результат:
* "Source": "/opt/sqb//config.env"  
* "Destination": "/app/config.env"
* "Mode": "ro"
* "RW": false
  
### 9. Валидация: контейнер НЕ root (UID согласован)

Вход в контейнер и проверка UID:
sudo docker exec -it cards-secure sh  id

Ожидаемо: UID соответствует сервисному пользователю на хосте (например 1001).

То же для payments:
sudo docker exec -it payments-secure sh  id

### 10. Валидация: конфиг читается, но не изменяется

Чтение:
cat /app/config.env

Проверка запрета записи (должно упасть с ошибкой):
echo "hack=true" >> /app/config.env

Результат:
* Permission denied или Read-only file system
    
Это подтверждает:

* конфиг защищён
    
* даже при компрометации контейнера злоумышленник не сможет модифицировать конфиг или внедрить изменения
    

Threat Mitigation (что именно снизили)
--------------------------------------

Данный baseline снижает риски:

*   lateral movement между сервисами (Cards ≠ Payments)
    
*   несанкционированного доступа к чувствительным конфигам
    
*   подмены конфигурации внутри контейнера
    
*   закрепления злоумышленника (persistence) через запись на диск контейнера
    

Итог (Result)
-------------

Реализованы принципы DevSecOps:

* **Least Privilege**: запуск без root и доступы по минимуму
    
* **Immutable Infrastructure**: --read-only делает файловую систему контейнера неизменяемой
    
* **Secure Configuration**: конфиг монтируется только для чтения
    
* **Reduced blast radius**: компрометация одного сервиса не даёт автоматический доступ ко второму
    

Работают контейнеры:
--------------
* cards-secure (sqb/cards-service:secure)
    
* payments-secure (sqb/payments-service:secure)
    
* portainer (UI для управления)




