### **System Design for a URL Shortening Service like Bitly**  

---

## **1. Functional Requirements**
1. **Shorten URL** – Users should be able to generate a short URL from a long URL.
2. **Redirect to Original URL** – When a short URL is accessed, it should redirect to the original URL.
3. **Custom Short URLs** – Users should have an option to specify custom aliases.
4. **Expiration & Deletion** – URLs should have an optional expiry time.
5. **User Accounts & Authentication** – Registered users can manage their shortened URLs. - (optional)
6. **Rate Limiting** – Prevent abuse by restricting excessive requests.

---

## **2. Non-Functional Requirements**
1. **Scalability** – The service should handle millions of URL shortening and redirections.
2. **Low Latency** – URL redirections should be extremely fast.
3. **High Availability** – The service should be highly available and resilient to failures.
4. **Security** – Prevent abuse (e.g., spamming, phishing) and ensure secure storage.
5. **Consistency vs. Availability Tradeoff** – Prioritize availability while ensuring eventual consistency.

---

## **3. Back-of-the-Envelope Estimation**
### **Assumptions:**
- **100M new short URLs per month** (~3.3M per day)
- **10:1 read-to-write ratio** (for every URL created, it is accessed 10 times)
- **1 billion requests per month**
- **Shortened URL size:** ~7 characters (Base62 encoding: `62^7 ≈ 3.5 trillion` unique URLs)
- **Data Storage:**
  - URL mapping (`short_url -> long_url`) ≈ 100 bytes per entry
  - **100M URLs per month → 10GB per month**
  - **1-year data → ~120GB**
  - **5 years of data → ~600GB**

---

## **4. Database Design**
### **Table: `urls`**
| Column        | Type            | Description |
|--------------|----------------|------------|
| `short_url`  | VARCHAR(10) (Primary Key) | Unique short URL ID |
| `long_url`   | TEXT            | Original long URL |
| `created_at` | TIMESTAMP       | Timestamp of creation |
| `expires_at` | TIMESTAMP (NULLABLE) | Expiration time |
| `user_id`    | VARCHAR(50) (Nullable) | Optional user who created it |


#### **Storage Choice:**
- **Primary DB**: **Amazon DynamoDB** (for fast key-value lookups)

---

## **5. API Design**
### **1. Shorten URL**
**Request:**  
```http
POST /shorten
Content-Type: application/json
{
    "long_url": "https://www.example.com/some-long-url",
    "custom_alias": "myalias",  // Optional
    "expires_at": "2025-12-31T23:59:59Z"
}
```
**Response:**
```json
{
    "short_url": "https://short.ly/abcd123"
}
```

### **2. Redirect URL**
**Request:**
```http
GET /:short-code
```
**Response:**
```json
{
    "long_url": "https://www.example.com/some-long-url"
}
```

and it will redirect to original site


---

## **6. High-Level Architecture with AWS Services**
### **Components:**
1. **API Gateway + Lambda** – Handle API requests (URL shortening, redirection).
2. **DynamoDB** – Key-value store for short URL mappings.
3. **ElastiCache (Redis)** – Cache hot URLs for faster redirects.
4. **CloudWatch** – Monitor API performance and errors.

### **Architecture Flow**
1. **User Requests Short URL → API Gateway → Lambda → DynamoDB**
2. **User Visits Short URL → API Gateway → Lambda**
   - If cached → Redirect immediately
   - Else → Lookup DynamoDB, update cache, redirect

---

## **7. Key Issues and Solutions**
### **1. How to Generate Unique Short URLs Efficiently?**
- Use **Base62 encoding** (`[A-Z, a-z, 0-9]`) to generate compact unique IDs.
- Maintain **sequential counters** (e.g., in DynamoDB with auto-incrementing keys).
- Use **hashing functions (SHA256)** to generate deterministic URLs.

### **2. Handling High Traffic and Scaling**
- **Use ElastiCache (Redis) for caching hot URLs** (reduce DB queries).
- **Distribute requests using AWS Global Accelerator**.
- **Use DynamoDB with on-demand capacity** for automatic scaling.

### **3. Preventing Abuse & Rate Limiting**
- **AWS WAF** to block suspicious traffic.
- **Rate limiting using API Gateway usage plans**.

### **4. Ensuring High Availability**
- **DynamoDB is multi-AZ replicated**.
- **CloudFront global caching** for serving static content.
- **Active-passive failover with Route 53**.

---


