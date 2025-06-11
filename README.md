# Model
# pubsub-choreography-with-deadline (시간초과 데드라인 설정 및 처리)
## 의도적인 딜레이를 통해 데드라인을 설정하여 주문 취소. 
https://labs.msaez.io/#/189596125/storming/509d668435970df20a437cccd0b6203f

![스크린샷 2025-06-11 120824](https://github.com/user-attachments/assets/879cd7df-2493-4f8e-8240-df2dbe4a5998)

## 터미널 작성 참고용
1. sdk install java
2. lombok 1.18.30으로 수정
- order, delivery, product, deadline 4개 다 수정해야함

3. kafka 모니터링 화면 띄우기
- cd kafka
- docker-compose exec -it kafka /bin/bash
- cd /bin
- ./kafka-console-consumer --bootstrap-server localhost:9092 --topic choreography.with.deadline

★ kafka consumer topic 확인하는 방법 application.yml을 통해 destination 설정된 값 확인

4. order 실행
- cd order/
- mvn spring-boot:run

5. delivery 실행
- cd delivery/
- mvn spring-boot:run

6. product 실행
- cd product/
- mvn spring-boot:run

7. deadline 실행
- cd deadline/
- mvn spring-boot:run

8. 데드라인 동작 확인
- http :8083/inventories productName=TV stock=1000   # id=1
- http :8083/inventories productName=RADIO stock=1000  # id=2
- http :8081/orders customerId=1 productId=1 productName=TV qty=10

- 1번 주문을 의도적으로 10초의 딜레이가 있음.
- order rejected 다음에 delivery Started로 넘어간다.

- {"eventType":"DeliveryStarted","timestamp":1749618405844,"orderId":"2","productId":"1","productName":"TV","qty":10,"customerId":"1","address":null,"status":null}
- {"eventType":"StockDecreased","timestamp":1749618405906,"id":1,"productName":"TV","productImage":null,"stock":980,"orderId":"2"}
- {"eventType":"OrderPlaced","timestamp":1749618405928,"id":2,"customerId":"1","customerName":null,"productId":"1","productName":"TV","qty":10,"address":null,"status":"APPROVED"}
- {"eventType":"DeliveryCancelled","timestamp":1749618406003,"orderId":"2","productId":"1","productName":"TV","qty":10,"customerId":"1","address":null,"status":null}
- {"eventType":"StockIncreased","timestamp":1749618406027,"id":1,"productName":"TV","productImage":null,"stock":990,"orderId":null}
- 무언가 동작은 되었지만 약간 꼬였다. 예방하고 싶다면 맥동성을 추가해서 처리하면 된다.



## Before Running Services
### Make sure there is a Kafka server running
```
cd kafka
docker-compose up
```
- Check the Kafka messages:
```
cd infra
docker-compose exec -it kafka /bin/bash
cd /bin
./kafka-console-consumer --bootstrap-server localhost:9092 --topic
```

## Run the backend micro-services
See the README.md files inside the each microservices directory:

- order
- inventory


## Run API Gateway (Spring Gateway)
```
cd gateway
mvn spring-boot:run
```

## Test by API
- order
```
 http :8088/orders id="id"productId="productId"qty="qty"customerId="customerId"amount="amount"status="status"address="address"
```
- inventory
```
 http :8088/inventories id="id"stock="stock"
```


## Run the frontend
```
cd frontend
npm i
npm run serve
```

## Test by UI
Open a browser to localhost:8088

## Required Utilities

- httpie (alternative for curl / POSTMAN) and network utils
```
sudo apt-get update
sudo apt-get install net-tools
sudo apt install iputils-ping
pip install httpie
```

- kubernetes utilities (kubectl)
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

- aws cli (aws)
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

- eksctl 
```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
```
