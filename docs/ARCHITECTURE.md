# 🏗️ Technical Architecture - Vishav

Comprehensive technical architecture for the Vishav ephemeral camera-first social media platform.

## System Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        CLIENT LAYER (Flutter)                   │
├─────────────────────────────────────────────────────────────────┤
│  • Camera Capture (Photo/Video)                                 │
│  • Real-time Filters & Effects                                 │
│  • JWT Authentication (Secure Storage)                         │
│  • WebSocket Client (Socket.io)                                │
│  • Encrypted Message Handling                                  │
└─────────────────────────────────────────────────────────────────┘
                              ↕
┌─────────────────────────────────────────────────────────────────┐
│                      API GATEWAY / REVERSE PROXY                │
├─────────────────────────────────────────────────────────────────┤
│  • HTTPS/TLS Termination                                        │
│  • Rate Limiting & DDoS Protection                             │
│  • Request Validation                                          │
│  • Load Balancing                                              │
└─────────────────────────────────────────────────────────────────┘
                              ↕
┌─────────────────────────────────────────────────────────────────┐
│                  APPLICATION LAYER (3 Options)                  │
├─────────────────┬──────────────────┬───────────────────────────┤
│  Node.js        │  Firebase        │  Python/Django            │
│  + Express      │  (Managed)       │  + FastAPI                │
│  + MongoDB      │  + Firestore     │  + PostgreSQL             │
│  + Socket.io    │  + Cloud Funcs   │  + Redis (Celery)         │
└─────────────────┴──────────────────┴───────────────────────────┘
                              ↕
┌─────────────────────────────────────────────────────────────────┐
│                    DATA & STORAGE LAYER                         │
├─────────────────────────────────────────────────────────────────┤
│  • Database (MongoDB/Firestore/PostgreSQL) - TTL Indexes       │
│  • File Storage (S3/Cloud Storage/MinIO)                       │
│  • Cache Layer (Redis)                                         │
│  • Message Queue (RabbitMQ/Kafka)                              │
└─────────────────────────────────────────────────────────────────┘
                              ↕
┌─────────────────────────────────────────────────────────────────┐
│                  EXTERNAL SERVICES                              │
├─────────────────────────────────────────────────────────────────┤
│  • Email Service (SendGrid)                                    │
│  • SMS Service (Twilio) - Optional                             │
│  • Analytics (Mixpanel/Amplitude)                              │
│  • Error Tracking (Sentry)                                     │
└─────────────────────────────────────────────────────────────────┘
```

---

## Technology Stack

### Frontend (Flutter)

```yaml
Dependencies:
  camera:               # Native camera integration
  image_picker:         # Gallery access
  socket_io_client:     # Real-time WebSocket
  jwt_decoder:          # JWT token parsing
  flutter_secure_storage: # Secure token storage
  encrypt:              # AES encryption
  image:                # Image processing & filters
  permission_handler:   # Camera/storage permissions
  path_provider:        # Local file storage
  provider:             # State management
  dio:                  # HTTP client
  
# Minimum Requirements:
SDK: Flutter 3.0+
Dart: 2.17+
iOS: 12.0+
Android: 21+
```

### Backend Option 1: Node.js + Express + MongoDB

```javascript
// package.json
{
  "dependencies": {
    "express": "^4.18.2",
    "mongodb": "^5.0.0",
    "mongoose": "^7.0.0",
    "socket.io": "^4.6.0",
    "jsonwebtoken": "^9.0.0",
    "bcryptjs": "^2.4.3",
    "dotenv": "^16.0.0",
    "cors": "^2.8.5",
    "helmet": "^7.0.0",
    "morgan": "^1.10.0",
    "express-rate-limit": "^6.6.0",
    "compression": "^1.7.4",
    "uuid": "^9.0.0",
    "crypto": "^1.0.1"
  },
  "devDependencies": {
    "nodemon": "^2.0.20",
    "jest": "^29.5.0",
    "supertest": "^6.3.0"
  }
}
```

### Backend Option 2: Firebase

```yaml
Services:
  Firebase Authentication:
    - Custom token generation
    - JWT verification
  
  Firestore:
    - Real-time database
    - TTL policies
    - Security rules
  
  Cloud Storage:
    - Media file storage
    - Auto-deletion policies
  
  Cloud Functions:
    - Serverless backend logic
    - Auto-delete expired messages
    - WebSocket fallback
  
  Cloud Pub/Sub:
    - Scheduled tasks
    - Event streaming
```

### Backend Option 3: Python + Django/FastAPI + PostgreSQL

```python
# requirements.txt
Django==4.2.0
djangorestframework==3.14.0
djangorestframework-simplejwt==5.2.0
channels==4.0.0  # WebSocket support
channels-redis==4.1.0
psycopg2-binary==2.9.6
celery==5.3.0
redis==4.5.0
python-decouple==3.8
cryptography==40.0.1
Pillow==9.5.0
django-cors-headers==3.14.0
gunicorn==20.1.0
whitenoise==6.4.0
```

---

## Deployment Architecture

### Development Environment

```
Local Machine
├── Flutter App (hot-reload)
├── Backend Server (localhost:3000 or 8000)
├── MongoDB/PostgreSQL (localhost)
└── Redis (localhost:6379)
```

### Staging Environment

```
AWS/GCP/Azure (Staging)
├── EC2 Instance (Backend)
│   ├── Node.js/Python app
│   └── Supervisor/PM2 process management
├── RDS (PostgreSQL) or Atlas (MongoDB)
├── ElastiCache Redis
├── S3/Cloud Storage
└── CI/CD Pipeline (GitHub Actions)
```

### Production Environment

```
AWS/GCP/Azure (Production)
├── Load Balancer (ALB/CLB)
│   ├── Auto Scaling Group
│   │   ├── EC2 Instance 1 (App)
│   │   ├── EC2 Instance 2 (App)
│   │   └── EC2 Instance N (App)
├── RDS (Multi-AZ) or Atlas (Sharded)
├── ElastiCache Redis Cluster
├── S3/GCS (CDN-enabled)
├── CloudFront/CDN
├── Route 53/Cloud DNS
├── CloudWatch Monitoring
├── AWS WAF (DDoS Protection)
└── Backup & Recovery Strategy

Kubernetes (Alternative):
├── Node Pool
│   ├── Pod 1 (App)
│   ├── Pod 2 (App)
│   └── Pod N (App)
├── StatefulSet (PostgreSQL)
├── Redis Operator
├── Ingress Controller
└── Helm Charts
```

---

## API Architecture

### REST Endpoints

```
Authentication:
  POST   /api/v1/auth/register      Register new user
  POST   /api/v1/auth/login         Login user
  POST   /api/v1/auth/logout        Logout user
  POST   /api/v1/auth/refresh       Refresh JWT token
  GET    /api/v1/auth/me            Get current user

Users:
  GET    /api/v1/users/{userId}     Get user profile
  PUT    /api/v1/users/{userId}     Update user profile
  GET    /api/v1/users/search       Search users
  POST   /api/v1/users/{userId}/avatar  Upload avatar

Messages:
  POST   /api/v1/messages           Send message
  GET    /api/v1/messages/{messageId}  Get message
  PUT    /api/v1/messages/{messageId}  Mark as viewed
  DELETE /api/v1/messages/{messageId}  Delete message
  GET    /api/v1/conversations/{userId} Get chat history

Stories:
  POST   /api/v1/stories            Create story
  GET    /api/v1/stories/{storyId}  Get story
  DELETE /api/v1/stories/{storyId}  Delete story
  GET    /api/v1/stories/feed       Get stories feed
  POST   /api/v1/stories/{storyId}/view  Record story view

Media:
  POST   /api/v1/media/upload       Upload media (image/video)
  GET    /api/v1/media/{mediaId}    Download media
  DELETE /api/v1/media/{mediaId}    Delete media
```

### WebSocket Events (Socket.io)

```javascript
// Client → Server
socket.emit('join_chat', { roomId: string })
socket.emit('typing', { roomId: string })
socket.emit('stop_typing', { roomId: string })
socket.emit('message_send', { roomId, content, mediaUrl })
socket.emit('screenshot_taken', { messageId: string })
socket.emit('story_viewed', { storyId: string })
socket.emit('user_online', { userId: string })
socket.emit('user_offline', { userId: string })

// Server → Client
socket.on('message_received', (message) => {})
socket.on('user_typing', (data) => {})
socket.on('user_stopped_typing', (data) => {})
socket.on('screenshot_notification', (data) => {})
socket.on('story_viewed_notification', (data) => {})
socket.on('user_come_online', (userId) => {})
socket.on('user_went_offline', (userId) => {})
socket.on('notification', (notification) => {})
```

---

## Security Architecture

### Authentication Flow

```
1. User Registration
   └─→ POST /api/v1/auth/register
       ├─ Hash password (bcrypt, 10 rounds)
       ├─ Generate RSA keypair (public/private)
       ├─ Store user in DB
       └─ Return JWT access token + refresh token

2. JWT Token Structure
   ├─ Header: { alg: "HS256", typ: "JWT" }
   ├─ Payload: { userId, username, iat, exp }
   └─ Signature: HMAC-SHA256(secret)

3. Token Validation
   ├─ Verify signature
   ├─ Check expiration
   ├─ Check token revocation list
   └─ Validate user still exists

4. Refresh Token Flow
   └─ POST /api/v1/auth/refresh
       ├─ Verify refresh token
       ├─ Check expiration
       ├─ Generate new access token
       ├─ Optional: rotate refresh token
       └─ Return new tokens
```

### Encryption Strategy

```
Messages (Content):
├─ Algorithm: AES-256-GCM
├─ Key Derivation: PBKDF2 (100,000 iterations)
├─ Plaintext: { content, mediaUrl, timestamp }
├─ Ciphertext stored in DB
└─ Decryption happens only on recipient device

Media Files (Image/Video):
├─ Algorithm: AES-256-CBC
├─ Stored in S3/Cloud Storage
├─ Server cannot decrypt (client-side encryption)
└─ TTL: Auto-delete after expiration

Authentication:
├─ Passwords: bcryptjs (cost factor 10)
├─ Tokens: HS256 (HMAC-SHA256)
└─ Keys: Rotated every 90 days
```

### Screenshot Detection

```
Flow:
1. Message sent with unique tracking ID
2. Server registers tracking ID in cache (Redis)
3. Client monitors screenshot (native API)
4. Screenshot detected → WebSocket event
5. Server broadcasts to sender
6. Sender notified with timestamp

Implementation:
├─ iOS: ScreenCaptureKit (iOS 17+)
├─ Android: ContentObserver on Screenshots folder
├─ Fallback: Client-side timestamp comparison
└─ Server-side: Record event in audit log
```

---

## Real-time Communication

### WebSocket Architecture

```
Server Setup:
├─ Socket.io namespace: /
├─ Room pattern: chat_{user1}_{user2}
├─ Event handlers: message, typing, online, etc.
├─ Broadcast to specific users/rooms
└─ Fallback: HTTP Long-polling

Message Queue:
├─ RabbitMQ for reliability
├─ Dead-letter queue for failed messages
├─ Message persistence
└─ Consumer replicas for scaling

Scaling WebSocket:
├─ Redis Adapter (socket.io-redis)
├─ Multiple Socket.io servers behind LB
├─ Session affinity / Sticky sessions
└─ Horizontal scaling support
```

### Real-time Flow

```
1. Client connects to WebSocket
   └─→ socket = io('https://api.vishav.com')
2. Authentication
   └─→ socket.auth = { token: jwtToken }
3. Join chat room
   └─→ socket.emit('join_chat', { roomId })
4. Send message
   └─→ socket.emit('message_send', { content, mediaUrl })
5. Server broadcasts
   └─→ socket.broadcast.to(roomId).emit('message_received', message)
6. Client receives and decrypts
   └─→ socket.on('message_received', (encrypted) => decrypt(encrypted))
```

---

## Performance Optimization

### Caching Strategy

```
Layer 1: Application Cache (Redis)
├─ User profiles (5 min TTL)
├─ Story feed (2 min TTL)
├─ Chat room metadata (1 hour TTL)
└─ Popular users (1 hour TTL)

Layer 2: Database Indexing
├─ User ID, Email
├─ Message: sender, recipient, created_at
├─ Story: user_id, created_at
├─ ExpireAt (TTL index)
└─ Compound indexes for sorting

Layer 3: CDN Caching
├─ Media files (images, videos)
├─ Static assets
├─ CloudFront/Akamai with 1 day TTL
└─ Invalidation on deletion

Layer 4: Browser Caching
├─ API responses (conditional)
├─ Static assets (1 year)
├─ Service Worker for offline support
└─ IndexedDB for local data
```

### Image/Video Optimization

```
Photos:
├─ Upload: Original + 3 thumbnails
│   ├─ Original (full resolution)
│   ├─ Preview (1280px)
│   ├─ Thumb (480px)
│   └─ Avatar (200px)
├─ Format: WEBP for web, HEIC for iOS
├─ Compression: 75-85% quality
└─ CDN delivery

Videos:
├─ Transcode to multiple bitrates
│   ├─ 480p (500 kbps)
│   ├─ 720p (1500 kbps)
│   └─ 1080p (3000 kbps)
├─ Format: H.264 + AAC audio
├─ Adaptive bitrate streaming (HLS)
└─ CDN delivery with playback optimization
```

### Database Query Optimization

```
N+1 Prevention:
├─ Eager load relationships (join/populate)
├─ Batch queries where possible
└─ Pagination (limit, offset)

Aggregation Pipeline:
├─ Stories feed aggregation
├─ User statistics calculation
├─ Trending stories
└─ Analytics queries

Query Caching:
├─ Frequently accessed queries in Redis
├─ Invalidate on data changes
└─ Stale cache strategy (5-60 min)
```

---

## Monitoring & Observability

### Metrics

```
Application Metrics:
├─ Request latency (p50, p95, p99)
├─ Error rate (5xx, 4xx)
├─ Active connections (WebSocket)
├─ Messages/sec throughput
└─ Cache hit rate

Infrastructure Metrics:
├─ CPU/Memory/Disk utilization
├─ Network I/O
├─ Database query latency
├─ Connection pool status
└─ Container health

Business Metrics:
├─ Daily Active Users (DAU)
├─ Monthly Active Users (MAU)
├─ Message sent per day
├─ Story creation rate
├─ Average session duration
└─ User retention rate
```

### Logging

```
Log Levels:
├─ DEBUG: Detailed app flow
├─ INFO: Important events
├─ WARNING: Recoverable issues
├─ ERROR: Failures requiring attention
└─ CRITICAL: System failures

Log Aggregation:
├─ ELK Stack (Elasticsearch, Logstash, Kibana)
└─ or CloudWatch (AWS)

Fields to log:
├─ Timestamp
├─ Log level
├─ Service name
├─ Request ID
├─ User ID
├─ Error stack trace
└─ Performance metrics
```

### Error Tracking

```
Sentry Configuration:
├─ Error reporting
├─ Stack trace analysis
├─ Release tracking
├─ Source maps
└─ Alert thresholds

Critical Alerts:
├─ Server down (5xx > 5%)
├─ Database unavailable
├─ WebSocket disconnects > 10%
├─ Payment processing failures
└─ Security incidents
```

---

## Scalability Strategy

### Horizontal Scaling

```
Phase 1 (0-10K Users):
├─ Single server + DB + Redis
├─ Basic monitoring
└─ Manual scaling

Phase 2 (10K-100K Users):
├─ 2-3 app servers + LB
├─ DB replication (read replicas)
├─ Redis cluster
├─ Automated scaling policies
└─ CDN for media

Phase 3 (100K-1M Users):
├─ Auto-scaling groups
├─ Database sharding
├─ Microservices (WebSocket, Auth, Media)
├─ Message queue (RabbitMQ)
├─ Dedicated cache layer
└─ Multi-region deployment

Phase 4 (1M+ Users):
├─ Kubernetes cluster
├─ Database federation
├─ Geo-distributed WebSocket servers
├─ Service mesh (Istio)
├─ Event streaming (Kafka)
└─ Global CDN with edge computing
```

### Database Scaling

```
Replication:
├─ Primary-Replica setup
├─ Read-heavy queries to replicas
├─ Write-heavy queries to primary
└─ Auto-failover on primary failure

Sharding:
├─ Shard by user_id
├─ Shard key: User ID (consistent hashing)
├─ 256 shards initial
├─ Resharding strategy for growth
└─ Cross-shard queries (aggregation)

Archival:
├─ Move old messages to archive DB
├─ Keep hot data in primary
├─ Archive after 30-90 days
└─ Separate storage tier (cheaper)
```

---

## Disaster Recovery

### Backup Strategy

```
Database:
├─ Hourly snapshots
├─ Daily full backup
├─ Off-site replication (different region)
├─ 30-day retention
└─ Test restore monthly

Media Storage:
├─ Cross-region replication
├─ Versioning enabled
├─ 7-day retention
└─ Immutable storage option

Configuration:
├─ Infrastructure as Code (Terraform)
├─ Version control (Git)
├─ Backup to S3
└─ Document runbooks
```

### Recovery Procedures

```
RTO (Recovery Time Objective): 1 hour
RPO (Recovery Point Objective): 15 minutes

Failover:
├─ Automatic on primary DB failure
├─ Manual approval for data corruption
├─ DNS failover to backup region
└─ Notification to ops team

Testing:
├─ Monthly disaster recovery drill
├─ Test restore from backup
├─ Validate data integrity
└─ Document findings
```

---

## Cost Optimization

### Infrastructure Costs

```
Compute (Monthly Estimate):
├─ App Servers: 2x t3.large = $60
├─ Database: RDS t3.large = $150
├─ Cache: ElastiCache t3.small = $30
├─ Load Balancer: ALB = $20
└─ Total: ~$260/month (for 100K users)

Storage (Monthly):
├─ Database: 100GB = $10
├─ Media: 1TB = $23 (S3)
├─ Backups: 500GB = $12
└─ Total: ~$45/month

Transfer (Monthly):
├─ Data out to internet = $0.09/GB
├─ Example: 100TB = $9,000
├─ CDN: 50% cheaper
└─ Total: ~$4,500/month (with CDN)

Reserved Instances:
├─ Save 40% on compute
├─ 1-year or 3-year commitment
└─ Recommended for prod
```

### Cost Reduction Strategies

```
✓ Use auto-scaling (pay for what you use)
✓ Archive old data (cheaper storage tier)
✓ Enable compression (reduce transfer)
✓ Use CDN (reduce egress costs)
✓ Spot instances (70% discount, trade reliability)
✓ Reserved instances (40% discount)
✓ Optimize queries (reduce database load)
✓ Cache aggressively (reduce DB queries)
```

---

## Development Workflow

### Git Workflow

```
Branching:
├─ main: Production-ready code
├─ develop: Integration branch
├─ feature/*: New features
└─ hotfix/*: Production fixes

Pull Request Process:
├─ Create feature branch
├─ Commit changes
├─ Open PR with description
├─ Code review (minimum 2 approvals)
├─ CI/CD tests pass
└─ Merge and deploy

Commit Convention:
├─ feat: New feature
├─ fix: Bug fix
├─ docs: Documentation
├─ style: Code style
├─ refactor: Code refactoring
├─ test: Tests
└─ chore: Build/dependencies
```

### CI/CD Pipeline

```
GitHub Actions Workflow:
├─ Trigger: Push to develop/main
├─ Stage 1: Lint & Format
│  ├─ ESLint / Black
│  └─ Prettier / Dartfmt
├─ Stage 2: Unit Tests
│  ├─ Jest / pytest
│  └─ Code coverage > 80%
├─ Stage 3: Integration Tests
│  ├─ API tests
│  └─ Database tests
├─ Stage 4: Build
│  ├─ Docker image build
│  └─ Push to registry
├─ Stage 5: Deploy (main only)
│  ├─ Deploy to staging
│  ├─ Run smoke tests
│  └─ Deploy to production
└─ Stage 6: Post-Deploy
   ├─ Health checks
   ├─ Performance monitoring
   └─ Alert on failures
```

---

## Conclusion

This architecture supports:
- ✅ 100K+ concurrent users
- ✅ Real-time message delivery
- ✅ Automatic message expiration
- ✅ Enterprise-grade security
- ✅ 99.9% uptime SLA
- ✅ Horizontal scalability
- ✅ Cost optimization

For questions or clarifications, refer to other documentation files or contact the team.
