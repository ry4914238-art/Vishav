# Flutter Frontend - Vishav

The Flutter frontend for the Vishav ephemeral camera-first social media platform.

## 📁 Project Structure

```
flutter_app/
├── lib/
│   ├── main.dart                  # App entry point
│   ├── constants/
│   │   └── colors.dart            # Design system (colors, typography, spacing)
│   ├── models/
│   │   ├── user.dart              # User model
│   │   ├── message.dart           # Message model (ephemeral)
│   │   └── story.dart             # Story model (24-hour auto-expire)
│   ├── services/
│   │   └── api_service.dart       # HTTP client with JWT auth
│   ├── providers/
│   │   ├── auth_provider.dart     # Authentication state
│   │   ├── chat_provider.dart     # Real-time WebSocket chat
│   │   └── story_provider.dart    # Stories feed management
│   ├── screens/
│   │   ├── camera_screen.dart     # Camera capture (TODO)
│   │   ├── chat_screen.dart       # 1-to-1 messaging (TODO)
│   │   └── stories_screen.dart    # 24-hour stories feed (TODO)
│   ├── widgets/
│   │   ├── camera_view.dart       # Camera preview widget (TODO)
│   │   ├── filter_overlay.dart    # Filter effects (TODO)
│   │   ├── message_bubble.dart    # Message UI component (TODO)
│   │   └── story_card.dart        # Story display card (TODO)
│   └── utils/
│       ├── encryption.dart        # AES-256 encryption (TODO)
│       └── permissions.dart       # Camera/storage permissions (TODO)
├── pubspec.yaml                   # Dependencies
└── README.md                       # This file
```

## 🚀 Getting Started

### Prerequisites

- Flutter 3.0+ and Dart 2.17+
- Xcode (for iOS) / Android Studio (for Android)
- Backend API running (see `../backend`)

### Installation

1. **Install dependencies:**
   ```bash
   cd flutter_app
   flutter pub get
   ```

2. **Configure backend URL:**
   Edit `lib/services/api_service.dart` and update `baseUrl`:
   ```dart
   static const String baseUrl = 'https://your-backend-url.com/api/v1';
   ```

3. **Run the app:**
   ```bash
   flutter run
   ```

## 🏗️ Architecture

### State Management (Provider)

The app uses **Provider** for state management with three main providers:

#### **AuthProvider**
- Handles user login/logout
- Manages JWT tokens (secure storage)
- Auto-refresh expired tokens
- User profile management

```dart
final authProvider = Provider.of<AuthProvider>(context);
bool isLoggedIn = authProvider.isLoggedIn;
User? user = authProvider.currentUser;
```

#### **ChatProvider**
- Real-time WebSocket connection (Socket.io)
- Ephemeral message handling (auto-delete)
- Typing indicators
- Screenshot detection notifications
- Conversation history

```dart
final chatProvider = Provider.of<ChatProvider>(context);
chatProvider.sendMessage(roomId, content: "Hello");
chatProvider.startTyping(roomId);
chatProvider.reportScreenshot(messageId);
```

#### **StoryProvider**
- Stories feed management
- 24-hour auto-expiration
- View tracking
- Story creation

```dart
final storyProvider = Provider.of<StoryProvider>(context);
await storyProvider.createStory(mediaUrl, mediaType: 'image');
await storyProvider.recordStoryView(storyId);
```

### API Integration

The **ApiService** handles all backend communication with:
- Automatic JWT token injection
- Token refresh on 401 responses
- Error handling and status code mapping
- Retry logic

```dart
final apiService = ApiService();
final user = await apiService.getCurrentUser();
final messages = await apiService.getChatHistory(userId);
```

## 🎨 Design System

### Colors

```dart
AppColors.primary        // #FF6B9D (Hot Pink)
AppColors.secondary      // #4A90E2 (Sky Blue)
AppColors.accent         // #00D4AA (Teal)
AppColors.error          // #EF4444 (Red)
AppColors.success        // #10B981 (Green)
```

### Typography

```dart
AppTypography.headline1   // 32px, Bold
AppTypography.headline2   // 28px, Bold
AppTypography.body        // 16px, Regular
AppTypography.caption     // 12px, Regular
```

### Spacing

```dart
AppSpacing.xs   // 4px
AppSpacing.sm   // 8px
AppSpacing.md   // 16px
AppSpacing.lg   // 24px
AppSpacing.xl   // 32px
```

## 📋 Data Models

### User
```dart
User(
  id: "user123",
  username: "john_doe",
  email: "john@example.com",
  avatar: "https://...",
  bio: "Camera enthusiast 📸",
  isVerified: true,
  createdAt: DateTime.now(),
)
```

### Message (Ephemeral)
```dart
Message(
  id: "msg123",
  senderId: "user1",
  recipientId: "user2",
  content: "Hey! 👋",
  mediaUrl: "https://...", // Optional
  mediaType: "text",
  viewed: false,
  screenshotTaken: false,
  createdAt: DateTime.now(),
  expiresAt: DateTime.now().add(Duration(hours: 24)),
)
```

### Story (24-Hour Auto-Expire)
```dart
Story(
  id: "story123",
  userId: "user1",
  mediaUrl: "https://...",
  mediaType: "image",
  caption: "Beautiful sunset 🌅",
  createdAt: DateTime.now(),
  expiresAt: DateTime.now().add(Duration(hours: 24)),
  views: [
    StoryView(userId: "user2", viewedAt: DateTime.now()),
  ],
)
```

## 🔐 Security Implementation

### JWT Token Management

```dart
// Automatic token refresh on 401
_dio.interceptors.add(
  InterceptorsWrapper(
    onError: (error, handler) async {
      if (error.response?.statusCode == 401) {
        await _refreshToken();
        return handler.resolve(await _retry(error.requestOptions));
      }
      return handler.next(error);
    },
  ),
);
```

### Secure Storage

Tokens are stored in platform-specific secure storage:
- **iOS**: Keychain
- **Android**: Android Keystore

```dart
final secureStorage = FlutterSecureStorage();
await secureStorage.write(key: 'access_token', value: token);
```

### Message Encryption (TODO)

Messages will be encrypted client-side using AES-256:

```dart
// To implement
final encrypted = encryptMessage(content, publicKey);
await apiService.sendMessage(recipientId, content: encrypted);
final decrypted = decryptMessage(encrypted, privateKey);
```

## 📱 Screen Architecture

### Navigation Flow (Swipe-Based)

```
┌─────────────────────────────────────┐
│   Chat (Left)                       │
│   - 1-to-1 Messages                 │
│   - Disappearing after view         │
├─────────────────────────────────────┤
│   Camera (Center - Main)            │
│   - Photo/Video Capture             │
│   - Live Filters                    │
│   - Flash, Switch Camera            │
├─────────────────────────────────────┤
│   Stories (Right)                   │
│   - 24-Hour Feed                    │
│   - Auto-Delete                     │
│   - View Tracking                   │
└─────────────────────────────────────┘
```

### Screen Components (TODO)

**CameraScreen**
- Camera preview with auto-startup
- Filter overlay system
- Capture button (photo/video toggle)
- Gallery thumbnail
- Flash toggle, camera switch

**ChatScreen**
- Conversation list with timestamps
- Real-time message updates
- Typing indicators
- Screenshot notifications
- Message expiration countdown

**StoriesScreen**
- Stories feed (24-hour)
- User avatars with story indicators
- View count & viewer list
- Story progress indicator
- Story creation UI

## 🎥 Camera Integration

### Permissions (TODO)

```dart
// Request camera and storage permissions
await Permission.camera.request();
await Permission.storage.request();
```

### Photo/Video Capture (TODO)

```dart
// Using camera package
final image = await CameraController.takePicture();
final video = await CameraController.startVideoRecording();
```

## 🔌 Real-Time Communication

### WebSocket Connection

```dart
// Connect to backend
socket.connect();
socket.emit('join_chat', {'roomId': 'chat_user1_user2'});

// Listen for events
socket.on('message_received', (message) {
  // Handle incoming message
});

// Send message in real-time
socket.emit('message_send', {
  'roomId': roomId,
  'content': 'Hello!',
});
```

### Events

```dart
// Client → Server
'join_chat'              // Join a chat room
'typing'                 // User typing
'stop_typing'            // User stopped typing
'message_send'           // Send message
'screenshot_taken'       // Screenshot notification
'story_viewed'           // Record story view
'user_online'            // User status
'user_offline'           // User status

// Server → Client
'message_received'       // New message
'user_typing'            // User typing indicator
'user_stopped_typing'    // Typing stopped
'screenshot_notification'// Screenshot detected
'story_viewed_notification' // Story viewed
'user_come_online'       // User online
'user_went_offline'      // User offline
```

## 📦 Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| camera | ^0.10.4 | Native camera integration |
| socket_io_client | ^2.0.2 | Real-time WebSocket |
| jwt_decoder | ^2.0.1 | JWT token parsing |
| flutter_secure_storage | ^9.0.0 | Secure token storage |
| encrypt | ^5.0.1 | AES encryption |
| provider | ^6.0.7 | State management |
| dio | ^5.3.1 | HTTP client |
| image_picker | ^1.0.2 | Gallery access |
| video_player | ^2.7.2 | Video playback |

## 🧪 Testing (TODO)

```bash
# Run unit tests
flutter test

# Run integration tests
flutter drive --target=test_driver/app.dart
```

## 📱 Building & Deployment

### Build APK (Android)

```bash
flutter build apk --release
# Output: build/app/outputs/apk/release/app-release.apk
```

### Build IPA (iOS)

```bash
flutter build ios --release
# Output: build/ios/ipa/
```

### Build Web (Experimental)

```bash
flutter build web --release
# Output: build/web/
```

## 🐛 Debugging

### Enable Debug Logs

```dart
// In main.dart
import 'dart:developer' as developer;

developer.log('Debug message', name: 'vishav');
```

### Hot Reload

```bash
flutter run
# Press 'r' for hot reload
# Press 'R' for hot restart
```

## 📚 Resources

- [Flutter Docs](https://flutter.dev/docs)
- [Provider Docs](https://pub.dev/packages/provider)
- [Dio HTTP Client](https://pub.dev/packages/dio)
- [Socket.io for Flutter](https://pub.dev/packages/socket_io_client)

## 🤝 Contributing

Follow these guidelines:
1. Create a feature branch: `git checkout -b feature/new-feature`
2. Commit changes: `git commit -m 'feat: add new feature'`
3. Push to remote: `git push origin feature/new-feature`
4. Open a Pull Request

## 📄 License

MIT License - See LICENSE file

---

**Next Steps:**
- [ ] Implement CameraScreen with photo/video capture
- [ ] Implement ChatScreen with message UI
- [ ] Implement StoriesScreen with feed
- [ ] Add encryption for messages
- [ ] Add camera filters and overlays
- [ ] Implement screenshot detection
- [ ] Add tests
- [ ] Deploy to App Store & Play Store
