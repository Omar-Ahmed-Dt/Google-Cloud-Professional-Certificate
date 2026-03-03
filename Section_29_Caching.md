---
tags:
  - gcp
---
## Overview
### Caching Use Cases
- Caching **infrequently** changing **data in database**
- Caching **infrequently** changing **dynamic content**
- Caching **user sessions** from applications  
- Caching **static content**

### TTL - Time To Live
- defines how long cached data remains valid before it is automatically expired and removed.
- After TTL expires: Cache entry is deleted and Next request fetches fresh data from source

---

## Memorystore
- Memorystore is Google Cloud’s fully managed in-memory cache service, mainly for Redis (and also Memcached).

**Memorystore = Managed Redis/Memcached**: 
- Memorystore for **Redis** (most common)
	- Persistent option
	- Supports Redis features (pub/sub, streams, TTL, sorted sets, etc.)
	- Great for sessions, caching, locks, rate limiting
	- You need persistence

- Memorystore for **Memcached**
	- Pure cache (no persistence)
	- Simple key-value caching
	- Good for web caching when you don’t need Redis features
	- If you want to create a session store, or if you want to cache database queries,
	- High performance
	- Memcached does not provide any persistence. If any of the nodes in a Memcached cluster fail, you lose the data which is present on it.

**Can be accessed from**:
- Compute Engine
- Google Kubernetes Engine
- Cloud Functions
- App Engine flexible and standard

> [!Note]
> **App Engine memcache service** 
> - legacy caching service in GCP
> - Temporary storage (not backed by persistent storage)

---

## Cloud CDN - Content Delivery Network
- Cloud CDN uses Google’s global edge network to deliver content with low latency worldwide.
- Instead of users hitting your server directly, they hit the nearest Google edge location.
- Result:
	- ⚡ Faster response
	- 🌎 Better global performance
	- 📉 Reduced backend load

- It Works With External HTTP(S) Load Balancer - Cloud CDN does NOT work alone: 
	- It integrates with External HTTP(S) Load Balancing
	- The Load Balancer:
		- Provides frontend IP address
		- Terminates HTTPS
		- Routes traffic
		- Connects to CDN cache layer

- The more frequently data changes → the lower the TTL is recommended
	- Max age for Static content: 72 hours and for Dynamic content: 5 minutes 

### Cache Key
```text
# https://shop.myapp.com/product/123?currency=usd

Protocol: https
Host: shop.myapp.com
Path: /product/123
Query: currency=usd
```

- A cache key is what CDN uses to decide: "Is this request the same as a previous one?"
	- If key matches → Cache Hit
	- If key differs → Cache Miss

- Default Cache Key Behavior: 
- By default, Cloud CDN uses:  `Full URI = Protocol + Host + Path + Query String`, Example: `https://yourwebsite.com/my-image/1.jpg` is not as `http://yourwebsite.com/my-image/1.jpg` , these are considered DIFFERENT , Cache miss (because http ≠ https).

#### Custom Cache Keys
- You can customize what parts of URL are used in cache key.
- You can tell CDN:
	- Ignore protocol
	- Ignore host
	- Ignore query string
	- Or include only specific query params

Example command:
```bash
gcloud compute backend-services update BACKEND_SERVICE \
  --enable-cdn \
  --no-cache-key-include-protocol \
  --no-cache-key-include-host \
  --no-cache-key-include-query-string
```

- `--enable-cdn`: Turns on Cloud CDN for that backend service. So now Requests go through Google Edge (GFE), Responses can be cached.
- `--no-cache-key-include-protocol`: http = https
- `--no-cache-key-include-host`: Ignore hostname in cache key, `https://www.example.com/image.jpg` = `https://example.com/image.jpg`
- `--no-cache-key-include-query-string`: Ignore everything after `?`, `/image.jpg` = `/image.jpg?utm=google` = `/image.jpg?tracking=123`
- Cache key = Path only. `https://store.com/product/5?currency=usd`, Cache key becomes `/product/5`

---

