Để setup thêm plugin mqtt trong rabbitmq.
Người dùng cần xác định trước để config thêm dòng command

```
/bin/bash -c \"rabbitmq-plugins enable --offline rabbitmq_mqtt rabbitmq_web_mqtt rabbitmq_amqp1_0; rabbitmq--server\"
```

và mở thêm port cho MQTT
Default: 1883

Dựa theo dòng command trên, plugin được thêm vào là: `rabbitmq_mqtt,rabbitmq_prometheus,rabbitmq_web_mqtt`

Để run rabbitmq có setup sẵn MQTT thì chạy docker-compose

```
docker compose -f rabbitmq-mqtt-plugin.yml up -d
```

Chúc may mắn
