Designing a **Sports Score Update System** like **Cricbuzz** requires handling **real-time updates**, **high traffic**, **scalability**, and **data consistency**.  

---

## **🔹 System Requirements**
### **1️⃣ Functional Requirements**
✅ Real-time match score updates  
✅ Support for multiple sports (Cricket, Football, etc.)  
✅ User subscription for score alerts  
✅ Live commentary, statistics, and match summaries  
✅ API support for third-party integrations  

### **2️⃣ Non-Functional Requirements**
✅ **Low latency:** Updates must be **real-time (<100ms delay)**  
✅ **High availability:** Must handle **millions of users** concurrently  
✅ **Scalability:** System should scale **horizontally**  
✅ **Event-driven architecture:** Efficient event handling for real-time updates  
✅ **Data consistency:** Ensure the latest scores are reflected correctly  

---

## **🔹 High-Level Architecture**
### **1️⃣ Components Overview**
1. **Data Ingestion Layer** (Fetch match data from stadiums, IoT devices, APIs)  
2. **Processing Layer** (Validate, transform, and store match data)  
3. **Streaming & Caching Layer** (Redis, Kafka for real-time updates)  
4. **API Gateway** (Serve requests via REST/GraphQL/WebSockets)  
5. **Frontend & Mobile Clients** (React, Flutter, etc.)  
6. **Monitoring & Logging** (Prometheus, Grafana, ELK stack)  

---

### **2️⃣ System Design Diagram**
```
                          +---------------------------+
                          |       Mobile Apps         |
                          |    Web & API Clients      |
                          +------------+--------------+
                                       |
                         +-------------v------------+
                         |       API Gateway        |
                         +-------------+------------+
                                       |
                  +----------------+  |  +----------------+
                  |  WebSocket API  |  |  |   REST API     |
                  +-------+--------+  |  +--------+-------+
                          |           |
          +---------------v-----------v------------------+
          |          Real-time Streaming (Kafka)         |
          +--------------+------------------------------+
                         |
      +------------------+------------------+
      |                                     |
+-----v-----+                         +-----v------+
|  Redis    |  <--- Caching Layer ---> |   DB       |
|  Cache    |                         | PostgreSQL  |
+-----------+                         +------------+
```

---

## **🔹 Database Schema (PostgreSQL)**
```sql
CREATE TABLE matches (
    match_id SERIAL PRIMARY KEY,
    team_a VARCHAR(50),
    team_b VARCHAR(50),
    match_status VARCHAR(20), -- LIVE, COMPLETED, UPCOMING
    start_time TIMESTAMP
);

CREATE TABLE scores (
    score_id SERIAL PRIMARY KEY,
    match_id INT REFERENCES matches(match_id),
    team VARCHAR(50),
    runs INT,
    wickets INT,
    overs FLOAT,
    last_updated TIMESTAMP DEFAULT NOW()
);
```

---

## **🔹 Key System Design Choices**
### **1️⃣ How to Ensure Real-Time Updates?**
- **Use WebSockets** for instant updates to mobile & web clients  
- **Push Notifications** via Firebase for alerts  
- **Kafka** to handle **real-time data streaming** from stadium sources  

### **2️⃣ How to Handle High Traffic?**
- **Load Balancers** (Nginx, AWS ALB)  
- **CDN (CloudFront, Akamai)** to reduce backend load  
- **Horizontal Scaling** (Kubernetes & Auto Scaling Groups)  

### **3️⃣ How to Reduce Database Load?**
- **Use Redis Cache** for frequently accessed data (latest score, player stats)  
- **Store only incremental updates** instead of full match data  

---

## **🔹 Tech Stack**
### **Backend**
- **Golang / Node.js / Java (Spring Boot)**
- **WebSockets / GraphQL for real-time updates**
- **Redis for caching live scores**
- **Kafka for event streaming**
- **PostgreSQL for storing match data**

### **Frontend**
- **React.js / Next.js** (Web)
- **Flutter / React Native** (Mobile)
- **Firebase Cloud Messaging (FCM) for push notifications**

### **Deployment**
- **Docker + Kubernetes** for container orchestration  
- **AWS/GCP/Azure** for cloud infrastructure  
- **Prometheus + Grafana** for monitoring  

---

## **🔹 API Endpoints**
### **1️⃣ Get Live Scores**
```http
GET /matches/live
Response:
{
    "match_id": 101,
    "team_a": "India",
    "team_b": "Australia",
    "score": {
        "team": "India",
        "runs": 250,
        "wickets": 4,
        "overs": 38.2
    }
}
```

### **2️⃣ Subscribe to Match Updates**
```http
POST /subscribe
Body:
{
    "user_id": 123,
    "match_id": 101,
    "notify": true
}
```

---

## **🔹 Scaling Strategies**
### **1️⃣ Handling 1 Million Concurrent Users**
- **Use WebSockets instead of Polling** for real-time updates  
- **Distribute Load with Kafka and Redis**  
- **Horizontal Scaling with Kubernetes**  

### **2️⃣ Handling Data Consistency**
- **Use Eventual Consistency** via Kafka Consumers  
- **Sharded Databases** for better performance  

---

## **🔹 Summary**
| Component      | Technology Used |
|---------------|----------------|
| **API Layer** | Golang, Node.js, Spring Boot |
| **Real-time Updates** | WebSockets, GraphQL, Kafka |
| **Database** | PostgreSQL (sharded) |
| **Caching** | Redis |
| **Scaling** | Kubernetes, Auto Scaling |
| **Monitoring** | Prometheus, Grafana |

This architecture **ensures real-time updates, handles scale efficiently, and provides a seamless user experience** just like Cricbuzz. 🚀  

#### Kafka event streaming



```
package main

import (
	"fmt"
	"log"
	"github.com/confluentinc/confluent-kafka-go/kafka"
)

type KafkaProducer struct {
	Producer *kafka.Producer
	Topic    string
}

func NewKafkaProducer(broker, topic string) (*KafkaProducer, error) {
	p, err := kafka.NewProducer(&kafka.ConfigMap{"bootstrap.servers": broker})
	if err != nil {
		return nil, err
	}
	return &KafkaProducer{Producer: p, Topic: topic}, nil
}

func (kp *KafkaProducer) PublishMessage(key, value string) error {
	deliveryChan := make(chan kafka.Event)
	err := kp.Producer.Produce(&kafka.Message{
		TopicPartition: kafka.TopicPartition{Topic: &kp.Topic, Partition: kafka.PartitionAny},
		Key:           []byte(key),
		Value:         []byte(value),
	}, deliveryChan)

	if err != nil {
		return err
	}

	e := <-deliveryChan
	m := e.(*kafka.Message)
	if m.TopicPartition.Error != nil {
		return fmt.Errorf("Delivery failed: %v", m.TopicPartition.Error)
	}

	close(deliveryChan)
	return nil
}

type KafkaConsumer struct {
	Consumer *kafka.Consumer
	Topic    string
}

func NewKafkaConsumer(broker, group, topic string) (*KafkaConsumer, error) {
	c, err := kafka.NewConsumer(&kafka.ConfigMap{
		"bootstrap.servers": broker,
		"group.id":          group,
		"auto.offset.reset": "earliest",
	})
	if err != nil {
		return nil, err
	}

	c.Subscribe(topic, nil)
	return &KafkaConsumer{Consumer: c, Topic: topic}, nil
}

func (kc *KafkaConsumer) ConsumeMessages() {
	for {
		msg, err := kc.Consumer.ReadMessage(-1)
		if err == nil {
			log.Printf("Received message: %s: %s", msg.Key, msg.Value)
			// Process match score update here
		} else {
			log.Printf("Consumer error: %v", err)
		}
	}
}

func main() {
	producer, err := NewKafkaProducer("localhost:9092", "match_scores")
	if err != nil {
		log.Fatalf("Failed to create Kafka producer: %v", err)
	}

	consumer, err := NewKafkaConsumer("localhost:9092", "score_group", "match_scores")
	if err != nil {
		log.Fatalf("Failed to create Kafka consumer: %v", err)
	}

	go consumer.ConsumeMessages()

	err = producer.PublishMessage("match1", "Team A 100/2 vs Team B 95/3")
	if err != nil {
		log.Printf("Failed to publish message: %v", err)
	}
}

```
