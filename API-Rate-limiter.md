### **Rate Limiting Algorithms Explained**  
Rate limiting is a technique used to **control the rate of incoming requests** to an API or service to prevent overuse, abuse, or system overload. Several algorithms are used for rate limiting:  

---

## **1️⃣ Token Bucket Algorithm**  
🔹 **Concept:**  
- A bucket holds a fixed number of tokens.  
- Each request **requires a token** to proceed.  
- Tokens are **added at a constant rate** (e.g., 10 tokens per second).  
- If no tokens are available, the request is **denied**.  

🔹 **Example:**  
- Suppose the bucket size is **10 tokens** and **replenishes 1 token per second**.  
- A user can make **up to 10 requests instantly**, then needs to wait for new tokens.  

🔹 **Use Case:**  
✅ Good for **bursty traffic** because it allows requests to be made in quick succession until tokens run out.  

---

## **2️⃣ Leaky Bucket Algorithm**  
🔹 **Concept:**  
- Requests enter a **bucket** (queue).  
- The bucket **leaks (processes) requests at a constant rate**.  
- If the bucket is **full**, new requests are **dropped**.  

🔹 **Example:**  
- If a system processes **5 requests per second**, but a user sends **100 requests instantly**, only **5 requests per second** will be processed, and extra requests **will be delayed or dropped**.  

🔹 **Use Case:**  
✅ Ensures a **steady request rate** and prevents **bursts of traffic** from overwhelming the system.  

---

## **3️⃣ Sliding Log Algorithm**  
🔹 **Concept:**  
- Stores a **timestamp** for each request in a **log**.  
- When a new request arrives, the system **removes outdated timestamps** beyond the time window and counts the remaining ones.  
- If the request count exceeds the limit, it’s **denied**.  

🔹 **Example:**  
- If a user is limited to **100 requests per minute**, the system checks the last **60 seconds** of logs.  
- If fewer than 100 requests exist, the new request is **allowed**.  

🔹 **Use Case:**  
✅ Provides **precise** rate limiting but requires more memory for storing logs.  

---

## **4️⃣ Sliding Window Counter Algorithm**  
🔹 **Concept:**  
- Divides time into **fixed windows** (e.g., 1-minute windows).  
- Counts requests in the **current and previous window** with a **weighted average** to smooth transitions.  

🔹 **Example:**  
- If the limit is **100 requests per minute**, but a request comes at **59 seconds**, the system considers a fraction of the previous window’s count.  

🔹 **Use Case:**  
✅ Reduces **abrupt cutoffs** seen in fixed window counters while keeping efficiency.  

---

## **5️⃣ Race Conditions in Distributed Systems**  
When **rate limiting is implemented across multiple servers**, race conditions can occur due to **inconsistent counters**. Solutions include:  
✔ **Centralized rate limiter** (single Redis instance).  
✔ **Consistent hashing** (assign requests to the same node).  
✔ **Distributed locks** (prevent race conditions).  

---

### **Comparison Table**
| Algorithm | Handles Bursts? | Memory Usage | Complexity | Use Case |
|-----------|---------------|--------------|------------|----------|
| **Token Bucket** | ✅ Yes | Low | Medium | Allows bursts, smooths traffic |
| **Leaky Bucket** | ❌ No | Low | Low | Smooth traffic flow |
| **Sliding Logs** | ✅ Yes | High | High | Precise tracking |
| **Sliding Window Counter** | ✅ Yes | Medium | Medium | Balanced approach |

Would you like **code examples** for any of these? 🚀
