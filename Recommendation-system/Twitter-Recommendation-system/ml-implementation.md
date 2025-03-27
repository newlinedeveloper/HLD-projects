## **🚀 ML Model Implementation for Twitter Recommendation System**  

We will implement a **hybrid recommendation system** combining **Collaborative Filtering, Content-Based Filtering, and Graph-Based Ranking** to provide personalized tweet recommendations.

---

## **🔹 1. Overview of the ML Model**
### **💡 Hybrid Approach**
1️⃣ **Collaborative Filtering** (User-User & Item-Item Similarity)  
2️⃣ **Content-Based Filtering** (NLP-based Tweet Analysis)  
3️⃣ **Graph-Based Ranking** (User-Tweet Interaction Graph)  
4️⃣ **Real-time Reinforcement Learning** (Multi-Armed Bandit for optimization)

---

## **🔹 2. Dataset & Features**
### **✅ Input Data Sources**
- **User Activity Logs:** Likes, Retweets, Comments, Follows
- **Tweet Metadata:** Tweet text, hashtags, timestamp
- **Graph Data:** Follower relationships, engagement graph

### **✅ Feature Engineering**
| Feature | Type | Description |
|---------|------|-------------|
| User-Tweet Interaction | Implicit | Likes, Retweets, Replies |
| User Graph Features | Explicit | Followers, Following Count |
| Tweet Embeddings | Text-based | TF-IDF, BERT embeddings |
| Trending Score | Real-time | Popularity score of tweets |
| User Interests | Derived | Top topics from tweets |

---

## **🔹 3. ML Model Implementation**

### **📌 Step 1: Data Preprocessing**
```python
import pandas as pd
import numpy as np
from sklearn.feature_extraction.text import TfidfVectorizer

# Load user-tweet interactions
user_tweet_df = pd.read_csv("user_engagements.csv")

# Load tweet data
tweets_df = pd.read_csv("tweets.csv")

# TF-IDF Vectorization for tweet content
vectorizer = TfidfVectorizer(max_features=5000)
tweet_embeddings = vectorizer.fit_transform(tweets_df['text']).toarray()
```

---

### **📌 Step 2: Collaborative Filtering (Matrix Factorization)**
```python
from surprise import SVD
from surprise import Dataset, Reader
from surprise.model_selection import train_test_split

# Prepare dataset for surprise library
reader = Reader(rating_scale=(0, 1))
data = Dataset.load_from_df(user_tweet_df[['user_id', 'tweet_id', 'engagement_score']], reader)

# Train-test split
trainset, testset = train_test_split(data, test_size=0.2)

# Train SVD Model
model = SVD()
model.fit(trainset)

# Predict for a new user-tweet pair
pred = model.predict(uid="user_123", iid="tweet_789")
print(pred.est)  # Predicted engagement score
```

---

### **📌 Step 3: Content-Based Filtering (Tweet Text Similarity)**
```python
from sklearn.metrics.pairwise import cosine_similarity

# Compute cosine similarity between tweets
tweet_similarity = cosine_similarity(tweet_embeddings)

# Get most similar tweets for a given tweet
def recommend_similar_tweets(tweet_id):
    idx = tweets_df[tweets_df['tweet_id'] == tweet_id].index[0]
    similar_scores = list(enumerate(tweet_similarity[idx]))
    similar_scores = sorted(similar_scores, key=lambda x: x[1], reverse=True)
    return [tweets_df.iloc[i[0]].tweet_id for i in similar_scores[1:6]]

print(recommend_similar_tweets("tweet_789"))
```

---

### **📌 Step 4: Graph-Based Ranking (PageRank for Engagements)**
```python
import networkx as nx

# Create a directed graph of user-tweet engagements
G = nx.DiGraph()

for _, row in user_tweet_df.iterrows():
    G.add_edge(row["user_id"], row["tweet_id"], weight=row["engagement_score"])

# Compute PageRank scores
pagerank_scores = nx.pagerank(G, alpha=0.85)

# Get top-ranked tweets
sorted_tweets = sorted(pagerank_scores.items(), key=lambda x: x[1], reverse=True)
top_tweets = [tweet for tweet, score in sorted_tweets[:10]]

print("Top trending tweets:", top_tweets)
```

---

### **📌 Step 5: Real-Time Reinforcement Learning (Multi-Armed Bandit)**
```python
import random

# Multi-Armed Bandit (Epsilon-Greedy) for exploring new tweets
class EpsilonGreedy:
    def __init__(self, epsilon=0.1):
        self.epsilon = epsilon
        self.rewards = {}

    def choose_tweet(self, tweets):
        if random.uniform(0, 1) < self.epsilon:
            return random.choice(tweets)  # Explore
        else:
            return max(tweets, key=lambda x: self.rewards.get(x, 0))  # Exploit

    def update_reward(self, tweet_id, reward):
        self.rewards[tweet_id] = self.rewards.get(tweet_id, 0) + reward

# Example usage
bandit = EpsilonGreedy()
tweet_pool = ["tweet_1", "tweet_2", "tweet_3"]
selected_tweet = bandit.choose_tweet(tweet_pool)
bandit.update_reward(selected_tweet, reward=1)
```

---

## **🔹 4. Deploying the Model in Production**
- **Batch Predictions:** Precompute recommendations and store in **Redis** for fast retrieval.
- **Real-Time Predictions:** Use **Kafka + Flink** to process new interactions and update recommendations.
- **API Endpoint:** Expose recommendations via a **Flask or FastAPI service**.

### **🚀 REST API for Serving Recommendations**
```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/recommendations', methods=['GET'])
def recommend():
    user_id = request.args.get('user_id')
    recommendations = get_user_recommendations(user_id)
    return jsonify({"user_id": user_id, "recommendations": recommendations})

def get_user_recommendations(user_id):
    # Fetch precomputed recommendations from Redis or compute in real-time
    return ["tweet_123", "tweet_456", "tweet_789"]

if __name__ == '__main__':
    app.run(debug=True)
```

---

## **🔹 5. Final Architecture**
### ✅ **Key Components**
🔹 **Data Processing:** Kafka + Spark for real-time tweet ingestion  
🔹 **Recommendation Engine:** Hybrid ML Model (Collaborative + Content-Based + Graph-Based)  
🔹 **Real-Time Ranking:** Reinforcement Learning (Multi-Armed Bandit)  
🔹 **Storage:** PostgreSQL for structured data, Redis for caching, ElasticSearch for search  
🔹 **Deployment:** FastAPI/Flask for serving recommendations  

---

## **🎯 Key Takeaways**
✔ **Hybrid Model (Collaborative + Content-Based + Graph-Based)**  
✔ **Scalable & Low Latency (<200ms response time)**  
✔ **Efficient Caching with Redis**  
✔ **Real-time Streaming (Kafka + Spark/Flink)**  
✔ **Optimized Recommendations using Reinforcement Learning**  
