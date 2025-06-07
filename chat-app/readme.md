# System Design for a Chat Application like WhatsApp
---

## ✅ 1. Functional Requirements

- One-on-one messaging
- Group messaging
- Real-time message delivery (online/offline users)
- Media support: images, videos, documents
- Message read/delivery status
- End-to-end encryption
- Message history (with optional deletion)
- Notifications (push, email)
- User profile, presence (online/offline)

---

## 🔒 2. Non-Functional Requirements

- High availability and low latency
- Horizontal scalability
- Consistency for messaging (eventual for presence, strong for message delivery)
- Security (encryption, authentication)
- Fault tolerance and reliability
- Compliance (e.g., GDPR)
- Rate limiting & abuse detection

---

## 📏 3. Back of the Envelope Estimation

### Assumptions:
- 100M daily active users
- Avg 50 messages/day/user = **5B messages/day**
- 300B messages/month
- Message size (avg): 1KB → 5TB/day raw text
- Media (10% users, avg 5MB/day) = 50TB/day
- Peak QPS: 100K messages/sec

### Storage:
- Text messages: 1PB/year
- Media storage: ~15–20PB/year (highly dependent on usage)

---

## 🧩 4. Database Design (NoSQL + SQL Hybrid)

### **Tables (SQL for metadata, NoSQL for messages)**

#### Users Table
| Field         | Type       |
|---------------|------------|
| user_id (PK)  | UUID       |
| name          | VARCHAR    |
| phone         | VARCHAR    |
| profile_photo | URL        |
| status        | TEXT       |
| last_seen     | TIMESTAMP  |

#### Conversations Table (1-1 or group)
| Field             | Type     |
|------------------|----------|
| conversation_id   | UUID     |
| is_group          | BOOLEAN  |
| group_name        | TEXT     |
| participants      | [UUIDs]  |

#### Messages (NoSQL — Cassandra or DynamoDB)
| Field            | Type       |
|------------------|------------|
| message_id       | UUID       |
| conversation_id  | UUID       |
| sender_id        | UUID       |
| content_type     | TEXT       |
| content          | TEXT/BLOB  |
| timestamp        | TIMESTAMP  |
| delivery_status  | ENUM       |

#### MessageStatus (optional for tracking per-user read)
| message_id  | user_id | status | timestamp |

---

## 🧪 5. API Design (REST + WebSocket)

### **Auth**
```http
POST /register
POST /login
```

### **Messaging**
```http
POST /messages/send            # for text/media
GET  /messages/:conversation   # fetch history (pagination)

WS /ws                         # socket for receiving real-time messages
```

### **Groups**
```http
POST /groups/create
POST /groups/:id/add
POST /groups/:id/remove
```

### **Media Upload**
```http
POST /media/upload             # returns a media URL
```

---

## 🚧 7. Key Issues & Solutions

| Issue                        | Solution                                                                 |
|-----------------------------|--------------------------------------------------------------------------|
| **Scalability**             | Stateless microservices, partitioned message storage (conversation_id)   |
| **Real-time delivery**      | WebSocket or MQTT based socket infrastructure, Kafka for fanout          |
| **Offline delivery**        | Store pending messages in DB or Redis, push on reconnect                 |
| **Message ordering**        | Use timestamp + vector clock / Snowflake ID                              |
| **Delivery/read status**    | Track status per message per user or use push notification receipts      |
| **Media handling**          | Upload → S3 → store signed URL in message                               |
| **End-to-end encryption**   | Client-side encryption using E2E (e.g., Signal protocol)                 |
| **Abuse/Spam prevention**   | Rate limiting + content filtering + user blocking                       |
| **Message deletion/privacy**| Soft/hard deletes, TTLs, GDPR compliance support                        |
| **Scaling sockets**         | Use load-balanced WS (via sticky sessions) + Redis pub/sub for messages  |

---

