# HƯỚNG DẪN SỬ DỤNG WEBSOCKET CHO CHAT & THÔNG BÁO MATCH

---

## 1. WebSocket là gì?

**WebSocket** là một giao thức mạng cho phép thiết lập kết nối hai chiều (full-duplex) giữa client và server trên một kết nối TCP duy nhất.  
Khác với HTTP (request-response), WebSocket cho phép server chủ động gửi dữ liệu về client bất cứ lúc nào, rất phù hợp cho các ứng dụng realtime như chat, thông báo, game...

**Ưu điểm:**
- Giao tiếp realtime, độ trễ thấp.
- Server chủ động push dữ liệu cho client.
- Tiết kiệm tài nguyên hơn so với polling.

---

## 2. Cấu hình endpoint WebSocket trên backend

**Spring Boot cấu hình endpoint WebSocket:**
```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {
    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        // Main endpoint
        registry.addEndpoint("/ws")
                .setAllowedOriginPatterns("*")
                .withSockJS();
        
        // Alternative endpoint without SockJS
        registry.addEndpoint("/ws")
                .setAllowedOriginPatterns("*");
        
        // Additional endpoints
        registry.addEndpoint("/websocket")
                .setAllowedOriginPatterns("*")
                .withSockJS();
        
        registry.addEndpoint("/websocket")
                .setAllowedOriginPatterns("*");
    }

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.enableSimpleBroker("/topic", "/queue", "/chat", "/notification");
        registry.setApplicationDestinationPrefixes("/app");
        registry.setUserDestinationPrefix("/user");
    }
}
```

**Cấu hình trong application.properties:**
```properties
# WebSocket configuration
app.websocket.endpoint=/ws
app.websocket.allowed-origins=http://localhost:3000,http://localhost:8100

# Server configuration
server.port=8080
server.servlet.context-path=/api
```

- **Endpoint client kết nối:** `/ws` (sẽ thành `/api/ws` với context-path)
- **Prefix subscribe:** `/topic` (broadcast), `/queue` (riêng tư), `/user/queue` (user-specific)
- **Prefix gửi message:** `/app`

---

## 3. Các khái niệm cơ bản

- **Endpoint:** Địa chỉ WebSocket server mà client sẽ kết nối (ví dụ: `ws://localhost:8080/api/ws`)
- **Subscribe:** Đăng ký lắng nghe một kênh (topic) để nhận dữ liệu realtime từ server.
- **Publish:** Gửi dữ liệu từ client lên server (thường qua các endpoint bắt đầu bằng `/app`).
- **Topic:** Kênh chung, nhiều client có thể cùng subscribe (ví dụ: `/topic/chat/1`).
- **Queue:** Kênh riêng tư, chỉ một client nhận (ví dụ: `/user/queue/notification`).

---

## 4. Cài đặt thư viện cho client

**Node.js:**
```bash
npm install @stomp/stompjs ws
```

**React/Web:**
```bash
npm install @stomp/stompjs sockjs-client
```

---

## 5. Kết nối WebSocket từ client

### Node.js Example
```js
const { Client } = require('@stomp/stompjs');
const WebSocket = require('ws');

const WS_URL = 'ws://localhost:8080/api/ws';
const jwtToken = 'YOUR_JWT_TOKEN';

const client = new Client({
  brokerURL: WS_URL,
  connectHeaders: {
    Authorization: `Bearer ${jwtToken}`,
  },
  webSocketFactory: () => new WebSocket(WS_URL),
  reconnectDelay: 5000,
  debug: (str) => console.log(str),
});

client.onConnect = () => {
  console.log('✅ WebSocket connected');
  // Đăng ký nhận tin nhắn phòng chat
  client.subscribe('/topic/chat/1', (message) => {
    const data = JSON.parse(message.body);
    console.log('Received:', data);
  });
};

client.activate();
```

### React Example (dùng SockJS)
```js
import { Client } from '@stomp/stompjs';
import SockJS from 'sockjs-client';

const WS_URL = 'http://localhost:8080/api/ws';
const jwtToken = localStorage.getItem('token');

const client = new Client({
  webSocketFactory: () => new SockJS(WS_URL),
  connectHeaders: {
    Authorization: `Bearer ${jwtToken}`,
  },
  reconnectDelay: 5000,
  debug: (str) => console.log(str),
});

client.onConnect = () => {
  client.subscribe('/topic/chat/1', (message) => {
    const data = JSON.parse(message.body);
    // Xử lý data
  });
};

client.activate();
```

---

## 6. Đăng ký nhận tin nhắn (subscribe)

Khi kết nối thành công, hãy subscribe vào topic phòng chat:
```js
client.subscribe('/topic/chat/{chatRoomId}', (message) => {
  const data = JSON.parse(message.body);
  // Xử lý các loại message: MESSAGE, TYPING, READ_RECEIPT, MESSAGE_RECALLED
});
```

---

## 7. Gửi tin nhắn (qua REST API)

```js
fetch('http://localhost:8080/api/chat/messages', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${jwtToken}`,
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    chatRoomId: 1,
    content: 'Hello!',
    messageType: 'TEXT',
  }),
});
```

**Các messageType được hỗ trợ:**
- `TEXT` - Tin nhắn văn bản
- `IMAGE` - Tin nhắn hình ảnh
- `AUDIO` - Tin nhắn âm thanh
- `VIDEO` - Tin nhắn video
- `FILE` - Tin nhắn file
- `EMOJI` - Tin nhắn emoji

> Tin nhắn sẽ được gửi tới tất cả client qua WebSocket sau khi backend xử lý.

---

## 8. Upload hình ảnh cho chat

### Upload hình ảnh
```js
const formData = new FormData();
formData.append('file', file);
formData.append('chatRoomId', chatRoomId);

fetch('http://localhost:8080/api/chat/upload-image', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${jwtToken}`,
  },
  body: formData,
})
.then(response => response.text())
.then(imageUrl => {
  // Gửi tin nhắn với hình ảnh
  fetch('http://localhost:8080/api/chat/messages', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${jwtToken}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      chatRoomId: chatRoomId,
      content: null,
      messageType: 'IMAGE',
      imageUrl: imageUrl, // URL trả về từ upload
    }),
  });
});
```

### Xóa hình ảnh (chỉ người upload mới xóa được)
```js
fetch('http://localhost:8080/api/chat/delete-image', {
  method: 'DELETE',
  headers: {
    'Authorization': `Bearer ${jwtToken}`,
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    imageUrl: 'path/to/image.jpg',
  }),
});
```

---

## 9. Gửi trạng thái "đang nhập" (typing)

### Client gửi typing indicator qua WebSocket
```js
// Khi user bắt đầu gõ
client.publish({
  destination: '/app/chat.typing',
  body: JSON.stringify({
    chatRoomId: 123,
    typing: true
  })
});

// Khi user ngừng gõ
client.publish({
  destination: '/app/chat.typing',
  body: JSON.stringify({
    chatRoomId: 123,
    typing: false
  })
});
```

### Server broadcast đến tất cả thành viên trong phòng
```json
{
  "type": "TYPING",
  "chatRoomId": 123,
  "senderId": 456,
  "content": "true", // hoặc "false"
  "timestamp": "2024-01-01T12:00:00"
}
```

### Client nhận và hiển thị
```js
client.subscribe('/topic/chat/123', function(message) {
    const data = JSON.parse(message.body);
    if (data.type === 'TYPING') {
        if (data.content === 'true') {
            showTypingIndicator(data.senderId);
        } else {
            hideTypingIndicator(data.senderId);
        }
    }
});
```

---

## 10. Đánh dấu tin nhắn đã đọc

```js
fetch('http://localhost:8080/api/chat/rooms/1/messages/read', {
  method: 'PUT',
  headers: {
    'Authorization': `Bearer ${jwtToken}`,
  },
});
```

> Sau khi gọi API này, backend sẽ gửi WebSocket message type `"READ_RECEIPT"` cho các client khác.

---

## 11. Thu hồi tin nhắn (Message Recall)

### Gửi yêu cầu thu hồi tin nhắn qua WebSocket
```js
client.publish({
  destination: '/app/chat.recallMessage',
  body: JSON.stringify({
    messageId: 123, // ID của tin nhắn cần thu hồi
  }),
});
```

### Gửi yêu cầu thu hồi qua REST API (fallback)
```js
fetch('http://localhost:8080/api/chat/messages/123/recall', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${jwtToken}`,
  },
});
```

### Xử lý thông báo tin nhắn bị thu hồi
```js
client.subscribe('/topic/chat/1', (message) => {
  const data = JSON.parse(message.body);
  switch (data.type) {
    case 'MESSAGE_RECALLED':
      // Cập nhật UI để hiển thị tin nhắn đã bị thu hồi
      updateMessageAsRecalled(data.messageId);
      break;
    // ... các case khác
  }
});
```

### Cấu trúc message thông báo thu hồi
```json
{
  "type": "MESSAGE_RECALLED",
  "chatRoomId": 1,
  "messageId": 123,
  "senderId": 456,
  "timestamp": "2024-01-01T12:00:00",
  "recalled": true,
  "recalledAt": "2024-01-01T12:00:00"
}
```

### Lưu ý về thu hồi tin nhắn
- **Thời gian thu hồi:** Chỉ có thể thu hồi tin nhắn trong vòng 30 phút sau khi gửi
- **Quyền thu hồi:** Chỉ người gửi tin nhắn mới có thể thu hồi tin nhắn của mình
- **Real-time:** Tất cả client trong phòng chat sẽ nhận được thông báo thu hồi ngay lập tức
- **Fallback:** Nên sử dụng REST API để xử lý lỗi tốt hơn

---

## 12. Xóa tin nhắn cho riêng mình

```js
fetch('http://localhost:8080/api/chat/messages/123/delete-for-me', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${jwtToken}`,
  },
});
```

> Tin nhắn sẽ bị ẩn khỏi danh sách tin nhắn của user này nhưng vẫn hiển thị cho user khác.

---

## 13. Xử lý các loại message từ WebSocket

```js
client.subscribe('/topic/chat/1', (message) => {
  const data = JSON.parse(message.body);
  switch (data.type) {
    case 'MESSAGE':
      // Hiển thị tin nhắn mới
      displayNewMessage(data);
      break;
    case 'TYPING':
      // Hiển thị "đang nhập..." cho user khác
      handleTypingIndicator(data);
      break;
    case 'READ_RECEIPT':
      // Đánh dấu tin nhắn đã được đọc
      markMessageAsRead(data);
      break;
    case 'MESSAGE_RECALLED':
      // Cập nhật UI để hiển thị tin nhắn đã bị thu hồi
      updateMessageAsRecalled(data.messageId);
      break;
    default:
      // Xử lý các loại khác nếu có
      console.log('Unknown message type:', data.type);
  }
});
```

---

## 14. Đăng ký nhận thông báo matching thành công (Match Notification)

### Đăng ký (subscribe) topic thông báo match

Khi user đăng nhập hoặc vào app, hãy subscribe vào topic thông báo cá nhân của user:

```js
// Subscribe vào user-specific notification queue
client.subscribe('/user/queue/notification', (message) => {
  const data = JSON.parse(message.body);
  if (data.type === 'MATCH') {
    // Xử lý thông báo match thành công
    alert(`Bạn đã match với ${data.content}!`);
    // Có thể tự động mở phòng chat hoặc cập nhật UI
  }
  // Có thể xử lý các loại notification khác ở đây
});
```

#### Cấu trúc message thông báo match (thực tế)
```json
{
  "id": "123",
  "type": "MATCH",
  "title": "New Match!",
  "content": "You and jane_smith have matched! Start chatting now!",
  "relatedEntityId": 5,
  "relatedEntityType": "MATCH",
  "timestamp": "2024-01-01T12:00:00",
  "action": "CREATE"
}
```

#### Luồng hoạt động
- Khi user A và user B cùng like nhau (match thành công), backend sẽ gửi thông báo qua WebSocket tới cả hai user.
- Client lắng nghe topic `/user/queue/notification` sẽ nhận được thông báo này ngay lập tức.

#### Gợi ý xử lý UI
- Hiển thị popup thông báo match thành công.
- Tự động chuyển sang giao diện chat với user vừa match (dùng `relatedEntityId` để lấy matchId).
- Cập nhật danh sách match hoặc chat room.

---

## 15. REST API Endpoints cho Chat

### Chat Rooms
- `GET /api/chat/rooms` - Lấy danh sách chat rooms với pagination
- `GET /api/chat/rooms/{chatRoomId}` - Lấy thông tin chat room
- `DELETE /api/chat/rooms/{chatRoomId}` - Deactivate chat room

### Messages
- `POST /api/chat/messages` - Gửi tin nhắn
- `GET /api/chat/rooms/{chatRoomId}/messages` - Lấy tin nhắn với pagination
- `PUT /api/chat/rooms/{chatRoomId}/messages/read` - Đánh dấu đã đọc
- `GET /api/chat/rooms/{chatRoomId}/messages/unread-count` - Số tin nhắn chưa đọc
- `POST /api/chat/messages/{messageId}/recall` - Thu hồi tin nhắn
- `POST /api/chat/messages/{messageId}/delete-for-me` - Xóa tin nhắn cho riêng mình

### File Upload
- `POST /api/chat/upload-image` - Upload hình ảnh cho chat
- `DELETE /api/chat/delete-image` - Xóa hình ảnh

### Notifications
- `GET /api/notifications` - Lấy danh sách thông báo
- `GET /api/notifications/unread` - Lấy thông báo chưa đọc
- `GET /api/notifications/unread/count` - Số thông báo chưa đọc
- `PUT /api/notifications/{notificationId}/read` - Đánh dấu đã đọc
- `PUT /api/notifications/read-all` - Đánh dấu tất cả đã đọc

---

## 16. Lưu ý bảo mật

- Luôn truyền JWT token khi kết nối và gọi API.
- Chỉ user thuộc phòng chat hoặc đúng userId mới nhận được tin nhắn/thông báo.
- WebSocket authentication được xử lý qua `WebSocketAuthInterceptor`.
- CORS được cấu hình cho các origin được phép.

---

## 17. Xử lý lỗi và reconnect

```js
client.onStompError = (frame) => {
  console.error('STOMP error:', frame);
  // Xử lý lỗi STOMP
};

client.onWebSocketError = (error) => {
  console.error('WebSocket error:', error);
  // Xử lý lỗi WebSocket
};

client.onWebSocketClose = () => {
  console.log('WebSocket connection closed');
  // Có thể thực hiện reconnect logic ở đây
};
```

---

## 18. Tham khảo thêm

- [@stomp/stompjs documentation](https://stomp-js.github.io/stomp-websocket/codo/extra/docs-src/Usage.md.html)
- [SockJS documentation](https://github.com/sockjs/sockjs-client)
- [Spring WebSocket documentation](https://docs.spring.io/spring-framework/reference/web/websocket.html)

---
