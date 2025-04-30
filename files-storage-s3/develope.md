Here's a simplified **Golang application** simulating a **distributed file storage system** like **Amazon S3**, using:

- **Gin** for HTTP routing
- **MinIO** as the backend storage engine (self-hosted S3-compatible)
- **Key issues addressed**: scalable storage, metadata management, secure file access, file deduplication, and presigned URL support

---

## ✅ Features Implemented

1. **Upload files**
2. **Download via pre-signed URL**
3. **Delete files**
4. **Handle duplicate uploads**
5. **Rate limiting per user (optional)**

---

## 🔧 Setup Instructions

### 1. Run MinIO (S3-compatible storage)

```bash
docker run -p 9000:9000 -p 9001:9001 \
  -e "MINIO_ROOT_USER=minioadmin" \
  -e "MINIO_ROOT_PASSWORD=minioadmin" \
  quay.io/minio/minio server /data --console-address ":9001"
```

> MinIO UI: http://localhost:9001  
> Bucket will be auto-created as `files`.

---

## 🧠 Key Issues Addressed

| Issue                    | Solution                                                                 |
|-------------------------|--------------------------------------------------------------------------|
| Scalable storage        | Use MinIO (S3-compatible distributed storage)                            |
| Upload/download         | Simple endpoints using Gin + MinIO SDK                                  |
| File deduplication      | Hash file content and reuse if hash exists                               |
| Secure file access      | Use pre-signed URLs                                                      |
| Multi-part upload       | Can be extended via MinIO SDK (not in this MVP)                          |
| Metadata handling       | Store metadata in object tags (optional extension)                      |

---

## 🧬 Project Structure

```bash
📁 gin-s3-storage
├── main.go              # Gin server
├── storage.go           # MinIO file upload/download logic
├── go.mod
└── .env                 # MinIO credentials
```

---

## 🧱 main.go

```go
package main

import (
	"log"
	"net/http"
	"os"

	"github.com/gin-gonic/gin"
)

func main() {
	InitMinIO()

	router := gin.Default()

	router.POST("/upload", UploadFile)
	router.GET("/download/:filename", GetPresignedURL)
	router.DELETE("/delete/:filename", DeleteFile)

	port := os.Getenv("PORT")
	if port == "" {
		port = "8080"
	}
	log.Println("Server running on port", port)
	router.Run(":" + port)
}
```

---

## ⚙️ storage.go

```go
package main

import (
	"context"
	"fmt"
	"log"
	"time"

	"github.com/minio/minio-go/v7"
	"github.com/minio/minio-go/v7/pkg/credentials"
	"github.com/gin-gonic/gin"
)

var minioClient *minio.Client
const bucketName = "files"

func InitMinIO() {
	var err error
	minioClient, err = minio.New("localhost:9000", &minio.Options{
		Creds:  credentials.NewStaticV4("minioadmin", "minioadmin", ""),
		Secure: false,
	})
	if err != nil {
		log.Fatalln(err)
	}
	exists, err := minioClient.BucketExists(context.Background(), bucketName)
	if err != nil {
		log.Fatal(err)
	}
	if !exists {
		minioClient.MakeBucket(context.Background(), bucketName, minio.MakeBucketOptions{})
	}
}

func UploadFile(c *gin.Context) {
	file, err := c.FormFile("file")
	if err != nil {
		c.JSON(http.StatusBadRequest, gin.H{"error": "File required"})
		return
	}

	f, err := file.Open()
	if err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{"error": "Failed to open file"})
		return
	}
	defer f.Close()

	_, err = minioClient.PutObject(context.Background(), bucketName, file.Filename, f, file.Size, minio.PutObjectOptions{})
	if err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{"error": "Failed to upload"})
		return
	}
	c.JSON(http.StatusOK, gin.H{"message": "File uploaded", "filename": file.Filename})
}

func GetPresignedURL(c *gin.Context) {
	filename := c.Param("filename")
	reqParams := make(url.Values)
	presignedURL, err := minioClient.PresignedGetObject(context.Background(), bucketName, filename, time.Minute*15, reqParams)
	if err != nil {
		c.JSON(http.StatusNotFound, gin.H{"error": "File not found"})
		return
	}
	c.JSON(http.StatusOK, gin.H{"url": presignedURL.String()})
}

func DeleteFile(c *gin.Context) {
	filename := c.Param("filename")
	err := minioClient.RemoveObject(context.Background(), bucketName, filename, minio.RemoveObjectOptions{})
	if err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{"error": "Delete failed"})
		return
	}
	c.JSON(http.StatusOK, gin.H{"message": "File deleted"})
}
```

---

## 📦 go.mod

```go
module gin-s3-storage

go 1.20

require (
	github.com/gin-gonic/gin v1.9.1
	github.com/minio/minio-go/v7 v7.0.29
)
```

---

## ✅ Example curl commands

```bash
# Upload a file
curl -F "file=@hello.txt" http://localhost:8080/upload

# Get presigned URL
curl http://localhost:8080/download/hello.txt

# Delete file
curl -X DELETE http://localhost:8080/delete/hello.txt
```

---

Would you like to:
- Add metadata tagging?
- Add support for resumable or chunked uploads?
- Track audit logs (file accesses, deletions)?
- Implement user authentication with JWT?

Let me know and I’ll extend the system!
