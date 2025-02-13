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


Here’s a **Golang implementation** of the **Sliding Window Counter** rate-limiting algorithm using **Redis** to handle distributed scenarios.  

This implementation ensures that API requests are limited **per user** within a **fixed window** while allowing smooth transitions between windows.  

---

### **🔹 Key Features of the Implementation**
✅ Uses **Golang** with **Redis** as a distributed rate limiter.  
✅ Tracks API requests per **user** using **time windows**.  
✅ Limits requests **per minute** with a **smoother transition** between windows.  

---

### **🔹 Golang Code for Sliding Window Counter Rate Limiter**
```go
package main

import (
	"fmt"
	"log"
	"time"

	"github.com/go-redis/redis/v8"
	"golang.org/x/net/context"
)

// Redis client
var ctx = context.Background()
var redisClient *redis.Client

// Rate limit parameters
const (
	WindowSize = 60               // Time window in seconds (1 minute)
	MaxRequests = 10              // Max requests allowed per user per window
)

// Initialize Redis client
func init() {
	redisClient = redis.NewClient(&redis.Options{
		Addr: "localhost:6379", // Redis address
	})
}

// Sliding window rate limiter function
func isRateLimited(userID string) bool {
	// Get the current timestamp (in seconds)
	currentTime := time.Now().Unix()

	// Define the Redis key
	key := fmt.Sprintf("rate_limit:%s", userID)

	// Remove outdated requests (older than the time window)
	redisClient.ZRemRangeByScore(ctx, key, "0", fmt.Sprintf("%d", currentTime-WindowSize))

	// Get the number of requests in the current window
	reqCount, err := redisClient.ZCard(ctx, key).Result()
	if err != nil {
		log.Println("Error getting request count:", err)
		return false
	}

	// If request count exceeds limit, block request
	if reqCount >= MaxRequests {
		return true
	}

	// Add new request with timestamp
	redisClient.ZAdd(ctx, key, &redis.Z{
		Score:  float64(currentTime),
		Member: fmt.Sprintf("%d", currentTime),
	})

	// Set expiration time for the key
	redisClient.Expire(ctx, key, time.Duration(WindowSize)*time.Second)

	return false
}

// Simulated API endpoint
func handleRequest(userID string) {
	if isRateLimited(userID) {
		fmt.Println("❌ Rate limit exceeded for user:", userID)
	} else {
		fmt.Println("✅ Request successful for user:", userID)
	}
}

func main() {
	// Simulate API calls from a user
	userID := "user_123"

	for i := 0; i < 12; i++ { // Try more than 10 requests
		handleRequest(userID)
		time.Sleep(1 * time.Second) // Simulate time gap between requests
	}
}
```

---

### **🔹 How It Works**
1. **Uses Redis Sorted Set (ZSET) to store timestamps** of requests per user.  
2. **Removes old requests** that fall outside the time window (e.g., 60 seconds).  
3. **Counts requests in the current window** before processing a new request.  
4. **Blocks requests if they exceed the limit** (e.g., 10 requests per minute).  
5. **Allows smooth transitions between time windows**.  

---

### **🔹 Example Output**
```plaintext
✅ Request successful for user: user_123
✅ Request successful for user: user_123
✅ Request successful for user: user_123
...
✅ Request successful for user: user_123
❌ Rate limit exceeded for user: user_123
❌ Rate limit exceeded for user: user_123
```

---

### **🔹 Why Redis for Rate Limiting?**
- ✅ **Fast & scalable** (handles millions of requests per second).  
- ✅ **Works in distributed systems** (multiple API servers share limits).  
- ✅ **Efficient memory usage** with TTL-based expiration.  

