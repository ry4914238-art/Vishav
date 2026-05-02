# 📊 Database Schema - Vishav

Complete database schemas for all three backend options with TTL (Time-To-Live) indexes for ephemeral message deletion.

## MongoDB Schema (Node.js + Express)

### Collections Overview

```
users
  ├─ _id (ObjectId)
  ├─ username (String, unique)
  ├─ email (String, unique)
  ├─ passwordHash (String, bcrypt)
  ├─ avatar (String, URL)
  ├─ bio (String)
  ├─ publicKey (String)
  ├─ isVerified (Boolean)
  ├─ createdAt (Date)
  ├─ updatedAt (Date)

messages
  ├─ _id (ObjectId)
  ├─ senderId (ObjectId, ref: users)
  ├─ recipientId (ObjectId, ref: users)
  ├─ content (String, encrypted AES-256)
  ├─ mediaUrl (String)
  ├─ mediaType (String: 'image' | 'video')
  ├─ viewed (Boolean)
  ├─ screenshotTaken (Boolean)
  ├─ createdAt (Date)
  ├─ viewedAt (Date)
  ├─ expiresAt (Date) ← TTL Index
  ├─ deletedAt (Date)

stories
  ├─ _id (ObjectId)
  ├─ userId (ObjectId, ref: users)
  ├─ mediaUrl (String)
  ├─ mediaType (String: 'image' | 'video')
  ├─ caption (String)
  ├─ createdAt (Date)
  ├─ expiresAt (Date) ← TTL Index (24h)
  ├─ views (Array of Objects)
  │  ├─ userId (ObjectId)
  │  ├─ viewedAt (Date)

chatRooms
  ├─ _id (ObjectId)
  ├─ participants (Array of ObjectIds)
  ├─ lastMessage (Object)
  │  ├─ text (String)
  │  ├─ sender (ObjectId)
  │  ├─ timestamp (Date)
  ├─ createdAt (Date)
  ├─ updatedAt (Date)

tokens
  ├─ _id (ObjectId)
  ├─ userId (ObjectId, ref: users)
  ├─ refreshToken (String)
  ├─ expiresAt (Date) ← TTL Index
  ├─ createdAt (Date)
```

### Detailed MongoDB Schemas

#### Users Collection

```javascript
db.createCollection("users", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["username", "email", "passwordHash"],
      properties: {
        _id: { bsonType: "objectId" },
        username: { 
          bsonType: "string",
          minLength: 3,
          maxLength: 30,
          pattern: "^[a-zA-Z0-9_.-]+$"
        },
        email: { 
          bsonType: "string",
          pattern: "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$"
        },
        passwordHash: { bsonType: "string" },
        avatar: { bsonType: "string" },
        bio: { 
          bsonType: "string",
          maxLength: 150
        },
        publicKey: { bsonType: "string" },
        isVerified: { bsonType: "bool" },
        createdAt: { bsonType: "date" },
        updatedAt: { bsonType: "date" }
      }
    }
  }
});

// Indexes
db.users.createIndex({ username: 1 }, { unique: true });
db.users.createIndex({ email: 1 }, { unique: true });
db.users.createIndex({ createdAt: -1 });
```

#### Messages Collection (with TTL)

```javascript
db.createCollection("messages", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["senderId", "recipientId", "createdAt"],
      properties: {
        _id: { bsonType: "objectId" },
        senderId: { bsonType: "objectId" },
        recipientId: { bsonType: "objectId" },
        content: { bsonType: "string" },
        mediaUrl: { bsonType: "string" },
        mediaType: { enum: ["text", "image", "video"] },
        viewed: { bsonType: "bool" },
        screenshotTaken: { bsonType: "bool" },
        createdAt: { bsonType: "date" },
        viewedAt: { bsonType: "date" },
        expiresAt: { bsonType: "date" },
        deletedAt: { bsonType: "date" }
      }
    }
  }
});

// Indexes
db.messages.createIndex({ senderId: 1, recipientId: 1, createdAt: -1 });
db.messages.createIndex({ recipientId: 1, viewed: 1 });

// TTL Index: Auto-delete after 24 hours (86400 seconds)
db.messages.createIndex(
  { expiresAt: 1 }, 
  { expireAfterSeconds: 0, background: true }
);
```

#### Stories Collection (with 24-hour TTL)

```javascript
db.createCollection("stories", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["userId", "mediaUrl", "createdAt"],
      properties: {
        _id: { bsonType: "objectId" },
        userId: { bsonType: "objectId" },
        mediaUrl: { bsonType: "string" },
        mediaType: { enum: ["image", "video"] },
        caption: { bsonType: "string" },
        createdAt: { bsonType: "date" },
        expiresAt: { bsonType: "date" },
        views: {
          bsonType: "array",
          items: {
            bsonType: "object",
            properties: {
              userId: { bsonType: "objectId" },
              viewedAt: { bsonType: "date" }
            }
          }
        }
      }
    }
  }
});

// Indexes
db.stories.createIndex({ userId: 1, createdAt: -1 });
db.stories.createIndex({ createdAt: 1 });

// TTL Index: Auto-delete after 24 hours from creation
db.stories.createIndex(
  { expiresAt: 1 }, 
  { expireAfterSeconds: 0, background: true }
);
```

#### Chat Rooms Collection

```javascript
db.createCollection("chatRooms", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["participants", "createdAt"],
      properties: {
        _id: { bsonType: "objectId" },
        participants: {
          bsonType: "array",
          items: { bsonType: "objectId" },
          minItems: 2,
          maxItems: 2
        },
        lastMessage: {
          bsonType: "object",
          properties: {
            text: { bsonType: "string" },
            sender: { bsonType: "objectId" },
            timestamp: { bsonType: "date" }
          }
        },
        createdAt: { bsonType: "date" },
        updatedAt: { bsonType: "date" }
      }
    }
  }
});

// Indexes
db.chatRooms.createIndex({ participants: 1, createdAt: -1 });
db.chatRooms.createIndex({ updatedAt: -1 });
```

---

## Firestore Schema (Firebase)

### Collections Overview

```
users/{uid}
  ├─ username (String)
  ├─ email (String)
  ├─ avatar (String)
  ├─ bio (String)
  ├─ publicKey (String)
  ├─ isVerified (Boolean)
  ├─ createdAt (Timestamp)
  ├─ updatedAt (Timestamp)

messages/{messageId}
  ├─ senderId (String, uid)
  ├─ recipientId (String, uid)
  ├─ content (String, encrypted)
  ├─ mediaUrl (String)
  ├─ mediaType (String)
  ├─ viewed (Boolean)
  ├─ screenshotTaken (Boolean)
  ├─ createdAt (Timestamp)
  ├─ viewedAt (Timestamp)
  ├─ expiresAt (Timestamp) ← Cloud Function deletes
  ├─ deletedAt (Timestamp)

stories/{storyId}
  ├─ userId (String, uid)
  ├─ mediaUrl (String)
  ├─ mediaType (String)
  ├─ caption (String)
  ├─ createdAt (Timestamp)
  ├─ expiresAt (Timestamp) ← Cloud Function deletes (24h)
  ├─ views (Map)
  │  ├─ {userId}: Timestamp

chatRooms/{roomId}
  ├─ participants (Array)
  ├─ lastMessage (Map)
  │  ├─ text (String)
  │  ├─ sender (String)
  │  ├─ timestamp (Timestamp)
  ├─ createdAt (Timestamp)
  ├─ updatedAt (Timestamp)
```

### Firestore Rules (with TTL)

```javascript
// firestore.rules
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Helper function: Check if user is authenticated
    function isAuthenticated() {
      return request.auth != null;
    }
    
    // Helper function: Check if user is document owner
    function isOwner(userId) {
      return request.auth.uid == userId;
    }
    
    // Users collection
    match /users/{userId} {
      allow read: if isAuthenticated();
      allow create: if isOwner(userId) && request.resource.data.keys().hasAll(['username', 'email']);
      allow update: if isOwner(userId);
      allow delete: if isOwner(userId);
    }
    
    // Messages collection
    match /messages/{messageId} {
      allow read: if isAuthenticated() && 
                      (isOwner(resource.data.senderId) || isOwner(resource.data.recipientId));
      allow create: if isAuthenticated() && 
                       isOwner(request.resource.data.senderId) &&
                       request.resource.data.keys().hasAll(['senderId', 'recipientId', 'content']);
      allow update: if isAuthenticated() && isOwner(resource.data.senderId);
      allow delete: if isAuthenticated() && 
                       (isOwner(resource.data.senderId) || isOwner(resource.data.recipientId));
    }
    
    // Stories collection
    match /stories/{storyId} {
      allow read: if isAuthenticated();
      allow create: if isAuthenticated() && isOwner(request.resource.data.userId);
      allow update: if isAuthenticated() && isOwner(resource.data.userId);
      allow delete: if isAuthenticated() && isOwner(resource.data.userId);
    }
    
    // Chat rooms collection
    match /chatRooms/{roomId} {
      allow read, write: if isAuthenticated() && 
                           request.auth.uid in resource.data.participants;
    }
  }
}
```

### Cloud Function: Auto-delete Expired Messages

```javascript
// functions/index.js - Firebase Cloud Functions
const functions = require('firebase-functions');
const admin = require('firebase-admin');

admin.initializeApp();

exports.deleteExpiredMessages = functions.pubsub
  .schedule('every 1 hours')
  .onRun(async (context) => {
    const db = admin.firestore();
    const now = admin.firestore.Timestamp.now();
    
    const snapshot = await db.collection('messages')
      .where('expiresAt', '<=', now)
      .where('deletedAt', '==', null)
      .limit(1000)
      .get();
    
    const batch = db.batch();
    
    snapshot.docs.forEach((doc) => {
      // Delete associated media from Storage
      if (doc.data().mediaUrl) {
        admin.storage().bucket()
          .file(doc.data().mediaUrl)
          .delete()
          .catch(err => console.log('File delete error:', err));
      }
      
      // Mark as deleted in DB
      batch.update(doc.ref, {
        deletedAt: admin.firestore.FieldValue.serverTimestamp()
      });
    });
    
    await batch.commit();
    console.log(`Deleted ${snapshot.docs.length} expired messages`);
  });

exports.deleteExpiredStories = functions.pubsub
  .schedule('every 2 hours')
  .onRun(async (context) => {
    const db = admin.firestore();
    const now = admin.firestore.Timestamp.now();
    
    const snapshot = await db.collection('stories')
      .where('expiresAt', '<=', now)
      .limit(1000)
      .get();
    
    const batch = db.batch();
    
    snapshot.docs.forEach((doc) => {
      // Delete media
      if (doc.data().mediaUrl) {
        admin.storage().bucket()
          .file(doc.data().mediaUrl)
          .delete()
          .catch(err => console.log('File delete error:', err));
      }
      
      // Delete document
      batch.delete(doc.ref);
    });
    
    await batch.commit();
    console.log(`Deleted ${snapshot.docs.length} expired stories`);
  });
```

---

## PostgreSQL Schema (Django/FastAPI)

### Django Models

```python
# models.py
from django.db import models
from django.contrib.auth.models import AbstractUser
from django.utils import timezone
from datetime import timedelta
import uuid

class CustomUser(AbstractUser):
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    username = models.CharField(max_length=30, unique=True)
    email = models.EmailField(unique=True)
    avatar = models.URLField(null=True, blank=True)
    bio = models.CharField(max_length=150, blank=True)
    public_key = models.TextField(null=True, blank=True)
    is_verified = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        ordering = ['-created_at']
        indexes = [
            models.Index(fields=['username']),
            models.Index(fields=['email']),
            models.Index(fields=['-created_at']),
        ]


class Message(models.Model):
    MESSAGE_TYPES = [
        ('text', 'Text'),
        ('image', 'Image'),
        ('video', 'Video'),
    ]
    
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    sender = models.ForeignKey(CustomUser, on_delete=models.CASCADE, related_name='sent_messages')
    recipient = models.ForeignKey(CustomUser, on_delete=models.CASCADE, related_name='received_messages')
    content = models.TextField(null=True, blank=True)  # Encrypted in app
    media_url = models.URLField(null=True, blank=True)
    media_type = models.CharField(max_length=10, choices=MESSAGE_TYPES, default='text')
    viewed = models.BooleanField(default=False)
    screenshot_taken = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)
    viewed_at = models.DateTimeField(null=True, blank=True)
    expires_at = models.DateTimeField(db_index=True)  # TTL index
    deleted_at = models.DateTimeField(null=True, blank=True)
    
    class Meta:
        ordering = ['-created_at']
        indexes = [
            models.Index(fields=['sender', 'recipient', '-created_at']),
            models.Index(fields=['recipient', 'viewed']),
            models.Index(fields=['expires_at']),
        ]
    
    def save(self, *args, **kwargs):
        if not self.expires_at:
            # Default: expire after 24 hours
            self.expires_at = timezone.now() + timedelta(hours=24)
        super().save(*args, **kwargs)


class Story(models.Model):
    MEDIA_TYPES = [
        ('image', 'Image'),
        ('video', 'Video'),
    ]
    
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    user = models.ForeignKey(CustomUser, on_delete=models.CASCADE, related_name='stories')
    media_url = models.URLField()
    media_type = models.CharField(max_length=10, choices=MEDIA_TYPES)
    caption = models.CharField(max_length=280, blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    expires_at = models.DateTimeField(db_index=True)  # 24 hours
    
    class Meta:
        ordering = ['-created_at']
        indexes = [
            models.Index(fields=['user', '-created_at']),
            models.Index(fields=['expires_at']),
        ]
    
    def save(self, *args, **kwargs):
        if not self.expires_at:
            # Expire after 24 hours
            self.expires_at = timezone.now() + timedelta(hours=24)
        super().save(*args, **kwargs)


class StoryView(models.Model):
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    story = models.ForeignKey(Story, on_delete=models.CASCADE, related_name='views')
    user = models.ForeignKey(CustomUser, on_delete=models.CASCADE)
    viewed_at = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        unique_together = ('story', 'user')
        indexes = [
            models.Index(fields=['story', 'viewed_at']),
        ]


class ChatRoom(models.Model):
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    participants = models.ManyToManyField(CustomUser, related_name='chat_rooms')
    last_message = models.TextField(null=True, blank=True)
    last_message_sender = models.ForeignKey(CustomUser, on_delete=models.SET_NULL, 
                                            null=True, blank=True, related_name='last_messages_sent')
    last_message_timestamp = models.DateTimeField(null=True, blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        ordering = ['-updated_at']
        indexes = [
            models.Index(fields=['participants', '-updated_at']),
            models.Index(fields=['-updated_at']),
        ]


class RefreshToken(models.Model):
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    user = models.OneToOneField(CustomUser, on_delete=models.CASCADE, related_name='refresh_token')
    token = models.TextField()
    expires_at = models.DateTimeField(db_index=True)
    created_at = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        indexes = [
            models.Index(fields=['expires_at']),
        ]
```

### PostgreSQL Triggers for TTL Deletion

```sql
-- Create function to delete expired messages
CREATE OR REPLACE FUNCTION delete_expired_messages()
RETURNS void AS $$
BEGIN
    DELETE FROM app_message
    WHERE expires_at <= NOW() AND deleted_at IS NULL;
END;
$$ LANGUAGE plpgsql;

-- Create trigger to run every hour
SELECT cron.schedule('delete_expired_messages', '0 * * * *', 'SELECT delete_expired_messages();');

-- Create function to delete expired stories
CREATE OR REPLACE FUNCTION delete_expired_stories()
RETURNS void AS $$
BEGIN
    DELETE FROM app_story
    WHERE expires_at <= NOW();
END;
$$ LANGUAGE plpgsql;

-- Create trigger to run every 2 hours
SELECT cron.schedule('delete_expired_stories', '0 */2 * * *', 'SELECT delete_expired_stories();');

-- Create indexes for TTL
CREATE INDEX idx_message_expires_at ON app_message(expires_at) WHERE deleted_at IS NULL;
CREATE INDEX idx_story_expires_at ON app_story(expires_at);
CREATE INDEX idx_refresh_token_expires_at ON app_refreshtoken(expires_at);
```

---

## Data Migration Guide

### Migrating Between Backends

All three backends follow the same logical schema. To migrate:

1. **Export data** from source backend
2. **Transform** to common JSON format
3. **Import** into target backend
4. **Verify** TTL indexes are working

Common fields across all backends:
- User ID, username, email, avatar, bio
- Message (senderId, recipientId, content, mediaUrl, viewed, expiresAt)
- Story (userId, mediaUrl, caption, expiresAt, views)
- ChatRoom (participants, lastMessage)

---

## Best Practices

✅ **Always use TTL indexes** for ephemeral data  
✅ **Encrypt sensitive fields** (passwords, messages) before storage  
✅ **Use transactions** for multi-document updates  
✅ **Index frequently queried fields** (userId, createdAt, expiresAt)  
✅ **Archive old messages** to separate collection before deletion  
✅ **Monitor disk space** for media storage  
✅ **Backup regularly** (daily minimum)  
✅ **Test TTL deletion** in staging before production  

