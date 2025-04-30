Creating a simplified **ride-sharing backend application** like Uber/Lyft using **Golang** and the `go-gin` framework is a great way to simulate core ride-sharing functionalities while addressing **key system design concerns** like **matching**, **geolocation**, and **concurrency**.

---

### ✅ **Key Features Covered**

- Rider requests ride (with location)
- Driver updates location
- System matches nearby drivers to riders
- Basic in-memory ride status tracking
- Rate limiting & API structure with `gin`

---

### 🧠 **Key Issues Addressed**

| Issue                             | Solution Implemented |
|----------------------------------|----------------------|
| Matching riders to nearby drivers | Haversine distance calculation |
| Real-time driver location updates | API for driver location push |
| Concurrency and consistency      | `sync.Mutex` for safe access |
| Rate limiting (to avoid abuse)   | Middleware using IP-based limiters |
| In-memory state for demo         | Fast prototyping (can swap with Redis/DB) |

---

## 🧾 Golang Ride-Sharing Backend (Gin + In-Memory)

```go
// main.go
package main

import (
	"log"
	"math"
	"net/http"
	"strconv"
	"sync"
	"time"

	"github.com/gin-gonic/gin"
	"golang.org/x/time/rate"
)

// Structs
type Location struct {
	Lat float64 `json:"lat"`
	Lng float64 `json:"lng"`
}

type RiderRequest struct {
	RiderID  string   `json:"rider_id"`
	Location Location `json:"location"`
}

type DriverLocationUpdate struct {
	DriverID string   `json:"driver_id"`
	Location Location `json:"location"`
}

// In-memory stores
var (
	drivers     = make(map[string]Location)
	riders      = make(map[string]Location)
	matches     = make(map[string]string) // rider_id -> driver_id
	lock        sync.Mutex
	rateLimiters = make(map[string]*rate.Limiter)
	rateLimit    = 10
	rateInterval = time.Minute
)

const (
	MAX_DISTANCE_KM = 5.0 // Max distance to match driver
)

// Helper: Calculate distance (Haversine)
func distanceKm(lat1, lon1, lat2, lon2 float64) float64 {
	const R = 6371 // Earth radius in KM
	dLat := (lat2 - lat1) * math.Pi / 180
	dLon := (lon2 - lon1) * math.Pi / 180

	a := math.Sin(dLat/2)*math.Sin(dLat/2) +
		math.Cos(lat1*math.Pi/180)*math.Cos(lat2*math.Pi/180)*
			math.Sin(dLon/2)*math.Sin(dLon/2)

	c := 2 * math.Atan2(math.Sqrt(a), math.Sqrt(1-a))
	return R * c
}

// Middleware: Basic rate limiting by IP
func rateLimiter() gin.HandlerFunc {
	return func(c *gin.Context) {
		ip := c.ClientIP()
		lock.Lock()
		limiter, exists := rateLimiters[ip]
		if !exists {
			limiter = rate.NewLimiter(rate.Every(rateInterval/time.Duration(rateLimit)), rateLimit)
			rateLimiters[ip] = limiter
		}
		lock.Unlock()

		if !limiter.Allow() {
			c.AbortWithStatusJSON(http.StatusTooManyRequests, gin.H{"error": "Rate limit exceeded"})
			return
		}
		c.Next()
	}
}

// POST /driver/update-location
func updateDriverLocation(c *gin.Context) {
	var update DriverLocationUpdate
	if err := c.ShouldBindJSON(&update); err != nil {
		c.JSON(http.StatusBadRequest, gin.H{"error": "Invalid payload"})
		return
	}
	lock.Lock()
	drivers[update.DriverID] = update.Location
	lock.Unlock()

	c.JSON(http.StatusOK, gin.H{"message": "Driver location updated"})
}

// POST /rider/request-ride
func riderRequestRide(c *gin.Context) {
	var request RiderRequest
	if err := c.ShouldBindJSON(&request); err != nil {
		c.JSON(http.StatusBadRequest, gin.H{"error": "Invalid payload"})
		return
	}

	lock.Lock()
	riders[request.RiderID] = request.Location

	// Find nearest driver
	minDist := MAX_DISTANCE_KM
	nearestDriver := ""

	for id, loc := range drivers {
		d := distanceKm(request.Location.Lat, request.Location.Lng, loc.Lat, loc.Lng)
		if d <= minDist {
			minDist = d
			nearestDriver = id
		}
	}

	if nearestDriver != "" {
		matches[request.RiderID] = nearestDriver
		delete(drivers, nearestDriver) // driver is now busy
	}
	lock.Unlock()

	if nearestDriver == "" {
		c.JSON(http.StatusOK, gin.H{"message": "No driver found nearby"})
	} else {
		c.JSON(http.StatusOK, gin.H{"message": "Driver matched", "driver_id": nearestDriver})
	}
}

// GET /ride/status/:rider_id
func getRideStatus(c *gin.Context) {
	riderID := c.Param("rider_id")
	lock.Lock()
	driverID, found := matches[riderID]
	lock.Unlock()

	if !found {
		c.JSON(http.StatusOK, gin.H{"status": "Not matched"})
	} else {
		c.JSON(http.StatusOK, gin.H{"status": "Matched", "driver_id": driverID})
	}
}

func main() {
	r := gin.Default()
	r.Use(rateLimiter())

	r.POST("/driver/update-location", updateDriverLocation)
	r.POST("/rider/request-ride", riderRequestRide)
	r.GET("/ride/status/:rider_id", getRideStatus)

	log.Println("🚖 Ride-sharing backend running on :8080")
	r.Run(":8080")
}
```

---

## 🧪 Sample API Tests

---

### 1. **Update Driver Location**
```bash
curl -X POST http://localhost:8080/driver/update-location \
  -H "Content-Type: application/json" \
  -d '{"driver_id":"D1", "location":{"lat":12.9716, "lng":77.5946}}'
```

---

### 2. **Request Ride as Rider**
```bash
curl -X POST http://localhost:8080/rider/request-ride \
  -H "Content-Type: application/json" \
  -d '{"rider_id":"R1", "location":{"lat":12.9721, "lng":77.5950}}'
```

---

### 3. **Check Ride Status**
```bash
curl http://localhost:8080/ride/status/R1
```

---

## 🔧 Improvements to Production-Grade System

| Area                   | Improvements |
|------------------------|--------------|
| **Persistence**        | Use Redis/PostgreSQL for state |
| **Real-time updates**  | Use WebSocket/gRPC |
| **Driver availability**| Add status field: available/busy |
| **Geospatial Indexing**| Use R-tree or PostGIS |
| **Authentication**     | JWT-based token support |
| **Scheduling algorithm** | Add ETA/traffic data |

---

Would you like a Postman collection or Dockerfile for this app too?
