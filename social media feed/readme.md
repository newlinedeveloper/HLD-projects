### **System Design for Social Media Feed (e.g., Facebook's News Feed)**

---
## **1. Functional Requirements**
- Users can create posts with text, images, and videos.
- Users can follow/unfollow other users.
- Users see a feed of posts from followed users.
- Posts are ranked based on relevance (e.g., engagement, recency, personalization).
- Users can like, comment, and share posts.
- Real-time updates for new posts and interactions.
- Infinite scrolling for continuous feed updates.

---
## **2. Non-Functional Requirements**
- **Scalability**: Should handle millions of users and posts.
- **Low Latency**: Feed should load in < 200ms.
- **High Availability**: System should not have downtime.
- **Consistency vs. Availability**: Use eventual consistency to improve performance.
- **Security**: Ensure user privacy and data protection.
- **Fault Tolerance**: Should handle server failures gracefully.

---
## **3. Back of the Envelope Estimation**
- **Active Users**: 500M daily active users (DAU).
- **Posts Per Day**: Each user posts ~2 times per day → 1B posts/day.
- **Feed Reads**: Each user fetches feed 10 times per day → 5B reads/day.
- **Storage**: Assuming an average post size of 1KB (text + metadata), storage required per day: **1TB/day**.
- **Bandwidth**: If each feed request fetches 20 posts, with 5B feed reads/day, that results in **100B posts fetched/day**.

---
## **4. Database Design**
### **Tables**

#### **Users Table**
```sql
CREATE TABLE users (
    user_id BIGINT PRIMARY KEY,
    username VARCHAR(255),
    email VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### **Posts Table**
```sql
CREATE TABLE posts (
    post_id BIGINT PRIMARY KEY,
    user_id BIGINT,
    content TEXT,
    media_url TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);
```

#### **Followers Table**
```sql
CREATE TABLE followers (
    follower_id BIGINT,
    following_id BIGINT,
    PRIMARY KEY (follower_id, following_id),
    FOREIGN KEY (follower_id) REFERENCES users(user_id),
    FOREIGN KEY (following_id) REFERENCES users(user_id)
);
```

#### **Likes Table**
```sql
CREATE TABLE likes (
    like_id BIGINT PRIMARY KEY,
    user_id BIGINT,
    post_id BIGINT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    FOREIGN KEY (post_id) REFERENCES posts(post_id)
);
```

#### **Comments Table**
```sql
CREATE TABLE comments (
    comment_id BIGINT PRIMARY KEY,
    user_id BIGINT,
    post_id BIGINT,
    content TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    FOREIGN KEY (post_id) REFERENCES posts(post_id)
);
```

---
## **5. API Design**

### **1. Create Post**
```
POST /posts
{
    "user_id": 123,
    "content": "This is a new post",
    "media_url": "https://cdn.example.com/image.jpg"
}
```

### **2. Get Feed**
```
GET /feed?user_id=123&limit=20
```
**Response:**
```
{
    "posts": [
        {"post_id": 1, "user_id": 456, "content": "Hello world", "likes": 100, "comments": 10},
        {"post_id": 2, "user_id": 789, "content": "New update!", "likes": 50, "comments": 5}
    ]
}
```

### **3. Like a Post**
```
POST /like
{
    "user_id": 123,
    "post_id": 456
}
```

### **4. Follow User**
```
POST /follow
{
    "follower_id": 123,
    "following_id": 456
}
```

---
## **6. High-Level Components with AWS Services**
### **1. API Layer**
- AWS **API Gateway**
- AWS **Lambda** (for serverless API processing) or **EC2** (if using monolithic service)
- AWS **Elastic Load Balancer** (for distributing traffic)

### **2. Storage Layer**
- **Amazon RDS (PostgreSQL/MySQL)**: Store structured data (users, posts, likes).
- **Amazon S3**: Store media content (images, videos).
- **Amazon DynamoDB**: Fast access for follower relationships.

### **3. Feed Generation & Ranking**
- AWS **Kinesis**: Process user activity logs in real-time.
- AWS **Lambda** + AWS **ElastiCache (Redis)**: Precompute feeds for active users.

### **4. Real-time Updates**
- AWS **WebSockets (via API Gateway)**: Send real-time notifications.
- AWS **SNS/SQS**: Handle async notifications.

### **5. Analytics & Monitoring**
- AWS **CloudWatch**: Monitor API performance.
- AWS **Athena**: Query logs and track engagement.

---
## **7. Addressing Key Issues & Solutions**

### **1. Feed Generation Strategy**
#### **Fan-out on Write (Push Model)** ✅
- When a user posts, we push it to their followers' precomputed feed stored in **Redis**.
- Works well for high-read, low-write systems (like Twitter).

#### **Fan-out on Read (Pull Model)** 🔄
- When a user loads their feed, we dynamically fetch posts from followed users.
- Works well for low-read, high-write systems.

💡 **Hybrid Approach:** Use a push model for active users and a pull model for inactive users.

### **2. Ranking & Personalization**
- Use **Machine Learning (AWS SageMaker)** to rank posts based on:
  - Recency
  - Engagement (likes/comments)
  - User interactions (past behavior)
  - Trending posts

### **3. Handling Hot Users (Celebrity Accounts)**
- Store their posts separately in **ElastiCache (Redis)** and use **CDN (CloudFront)** for fast delivery.

### **4. Real-time Engagement Updates**
- Use **WebSockets** (API Gateway) for real-time like/comments count updates.

### **5. Rate Limiting & Security**
- Implement **AWS WAF** to prevent abuse.
- Use **IAM Roles & Cognito** for authentication.
- Implement **API Gateway Rate Limiting**.

---
## **Conclusion**
- A **scalable, real-time, low-latency** feed system requires a mix of **SQL, NoSQL, caching, event processing, and ranking models**.
- **AWS services** like RDS, DynamoDB, ElastiCache, Kinesis, and S3 help scale efficiently.
- **Hybrid fan-out strategies** ensure an optimized feed experience.
- **Machine learning models** enhance engagement ranking.

🚀 **Final Thoughts:** The system should balance **performance, scalability, and personalization** while ensuring **high availability** and **low latency**.

