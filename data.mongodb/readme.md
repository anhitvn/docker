## mongo docker

---

## Một số command mongo thường dùng

Chạy lại và cài đè lên container đã tồn tại
docker compose -f docker-compose.yaml up -d mongo --force-recreatecat

docker compose -f docker-compose.yaml down mongo
docker compose -f docker-compose.yaml up -d mongo

docker exec -it mongo /bin/bash

===
mongorestore -h localhost:27017 -u admin -d erp_production --authenticationDatabase admin /data/backup/erp_production

---

mongo -h localhost:27017 -u admin -p `<password> `--authenticationDatabase admin
use erp_production

db.createUser({
  user: "erp_production",
  pwd: "`<password>`",
  roles: [ { role: "dbOwner", db: "erp_production" } ]
})
db.getUsers()

mongosh --host localhost --port 27017 -u admin -p `<password>`




## Setup trực tiếp trên ubuntu 18.04. (Không dùng docker)

---

Show all database

step 1: Login mongo: `mongo --host 192.168.45.227 --port 27017 -u [user] -p "[Password]"`

use command mongo cli:

```
> show dbs;
READ__ME_TO_RECOVER_YOUR_DATA  0.000GB
admin                          0.000GB
config                         0.000GB
erp_production                 0.004GB
erp_staging                    0.003GB
local  show dbs;
```

### Lệnh backup:

```
mongodump --host 127.0.0.1
    --port 27017
    --db erp_production
    --username administrator
    --authenticationDatabase admin
    --out /backup/erp_production/$(date +%Y%m%d)
```

### Lệnh khôi phục:

```
mongorestore -h 127.0.0.1:27017 -u administrator -d erp_production --authenticationDatabase admin /backup/erp_production/20250403
```

====

Để sử dụng tính năng MongoDB Replication và đồng bộ dữ liệu từ VPS 1 (192.168.45.227) sang VPS 2 (192.168.45.53), bạn cần thực hiện các bước sau:

**1. Cấu hình Replica Set trên VPS 1 (192.168.45.227):**

* **Chỉnh sửa file cấu hình MongoDB (`/etc/mongod.conf`):**

  **YAML**

  ```
  replication:
    replSetName: "myReplSet"
  net:
    bindIp: 192.168.45.227,127.0.0.1
  ```

  * `replSetName`: Đặt tên cho Replica Set (ví dụ: "myReplSet").
  * `bindIp`: Chỉ định IP mà MongoDB sẽ lắng nghe (bao gồm IP của VPS 1).
* **Khởi động lại MongoDB:**

  **Bash**

  ```
  sudo systemctl restart mongod
  ```
* **Khởi tạo Replica Set:**

  **Bash**

  ```
  mongo --host 192.168.45.227 --port 27017
  rs.initiate()
  ```

**2. Cấu hình Replica Set trên VPS 2 (192.168.45.53):**

* **Chỉnh sửa file cấu hình MongoDB (`/etc/mongod.conf`):**

  **YAML**

  ```
  replication:
    replSetName: "myReplSet"
  net:
    bindIp: 192.168.45.53,127.0.0.1
  ```

  * `replSetName`: Phải giống với tên Replica Set trên VPS 1 ("myReplSet").
  * `bindIp`: Chỉ định IP mà MongoDB sẽ lắng nghe (bao gồm IP của VPS 2).
* **Khởi động lại MongoDB:**

  **Bash**

  ```
  sudo systemctl restart mongod
  ```
* **Thêm VPS 2 vào Replica Set:**

  **Bash**

  ```
  mongo --host 192.168.45.227 --port 27017
  rs.add("192.168.45.53:19105")
  ```

  * Lưu ý: Vì VPS 2 sử dụng port NAT 19105, bạn cần chỉ định port này.

**3. Kiểm tra trạng thái Replica Set:**

* Trên VPS 1 hoặc VPS 2:

  **Bash**

  ```
  mongo --host <IP_VPS> --port <PORT_VPS>
  rs.status()
  ```

  * Thay `<IP_VPS>` và `<PORT_VPS>` bằng IP và port của một trong hai VPS.
  * Lệnh này sẽ hiển thị trạng thái của Replica Set, bao gồm các thành viên và trạng thái đồng bộ hóa.

**Lưu ý:**

* **Port NAT:** Đảm bảo rằng port NAT 19105 trên VPS 2 được cấu hình chính xác và có thể truy cập từ VPS 1.
* **Tường lửa:** Kiểm tra tường lửa trên cả hai VPS để đảm bảo rằng các port 27017 và 19105 được mở.
* **Xác thực:** Nếu MongoDB của bạn được cấu hình với xác thực, bạn cần cung cấp thông tin xác thực khi thêm thành viên vào Replica Set.
* **Đồng bộ hóa ban đầu:** Quá trình đồng bộ hóa ban đầu có thể mất một khoảng thời gian, tùy thuộc vào kích thước dữ liệu.
* **Kiểm tra liên tục:** Sử dụng `rs.status()` để kiểm tra trạng thái đồng bộ hóa thường xuyên.

**Ví dụ cấu hình hoàn chỉnh (VPS 1):**

**YAML**

```
replication:
  replSetName: "myReplSet"
net:
  bindIp: 192.168.45.227,127.0.0.1
storage:
  dbPath: /var/lib/mongodb # Đường dẫn dữ liệu MongoDB
```

**Ví dụ cấu hình hoàn chỉnh (VPS 2):**

**YAML**

```
replication:
  replSetName: "myReplSet"
net:
  bindIp: 192.168.45.53,127.0.0.1
storage:
  dbPath: /var/lib/mongodb # Đường dẫn dữ liệu MongoDB
```

Bằng cách thực hiện các bước trên, bạn sẽ thiết lập thành công MongoDB Replication và đồng bộ dữ liệu giữa hai VPS.
