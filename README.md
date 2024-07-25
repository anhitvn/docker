# Docker compose with redis-rabbitmq-manage

Đầu tiên, tôi sẽ để link được thao khảo tại đây [Link](https://gist.github.com/ps-jessejjohnson/cbe0df09431fb59ab3870dba5c583fa0)

Dù các bạn không biết nhưng rất cảm ơn repo/comment của các bạn, nó rất hữu ích cho tôi:

- @ps-jessejjohnson
- @ps-przemekaugustyn

Với docker-compose.yml hiện tại đã có thể để cho rabbitmq và redis hoạt động.

**Step 1:** git clone resource

```
git clone https://github.com/anhitvn/docker-redis-rabbitmq.git
cd. docker-redis-rabbitmq
```

**Step 2:** Tôi đã bổ sung thêm var trong rabbitmq để dễ hơn trong tạo account cho rabbitmq.

Bạn cần thay username/password bạn muốn vào trong docker-compose > rabbitmq

```
...
    environment:
      - RABBITMQ_DEFAULT_USER=admin
      - RABBITMQ_DEFAULT_PASS=your_password
```

Tương tự trong app redis bạn cũng cần thay password redis tương ứng.

```
...
    environment:
      - REDIS_PASSWORD=your_password
```

**Một số thay đổi trong redis.conf và lưu ý quan trọng.**

Config để cho phép các network truy cập redis.

```
bind 0.0.0.0
```

Config yêu cầu nhập password (Config default)

```
protected-mode yes
```

Config password default

```
# requirepass foobared
requirepass your_password
```

Cấu hình maxmemory:

```
# maxmemory <bytes>
maxmemory 4gb
```

Điều này giới hạn Redis chỉ sử dụng tối đa 4GB RAM.

Kiểm soát giới hạn truy cập

```
# maxclients 10000
maxclients 96
```

Cấu hình maxmemory-policy:

```
# maxmemory-policy noeviction
maxmemory-policy volatile-lru
```

Chính sách này sẽ xóa các key "least recently used" (LRU) khi Redis vượt quá giới hạn maxmemory.

Tùy chọn, bạn cũng có thể cấu hình maxmemory-samples:

```
# maxmemory-samples 5 => Tôi không dùng config này trong tài liệu này
```

Điều này xác định số lượng key được mẫu khi tìm kiếm key "least recently used" (LRU). Giá trị càng cao thì độ chính xác càng tốt, nhưng hiệu suất sẽ giảm.

**Step 3:** Start container

```
docker-compose up
```

**Step 4:** Test lại redis connection với redis-cli

Máy tôi đã setup redis-cli sẵn, nếu bạn chưa setup hãy setup redis-cli mới dùng được comment này.

```
redis-cli -h [your ip] -p 16379 -a 'your_password' ping
```
