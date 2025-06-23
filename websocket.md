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
        registry.addEndpoint("/api/ws") // endpoint cho client kết nối
                .setAllowedOrigins("*")
                .withSockJS();
    }

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.enableSimpleBroker("/topic", "/queue"); // các prefix cho subscribe
        registry.setApplicationDestinationPrefixes("/app"); // prefix cho client gửi message
    }
}
```
- **Endpoint client kết nối:** `/api/ws`
- **Prefix subscribe:** `/topic` (broadcast), `/queue` (riêng tư)
- **Prefix gửi message:** `/app`

---

## 3. Các khái niệm cơ bản

- **Endpoint:** Địa chỉ WebSocket server mà client sẽ kết nối (ví dụ: `ws://localhost:8080/api/ws`)
- **Subscribe:** Đăng ký lắng nghe một kênh (topic) để nhận dữ liệu realtime từ server.
- **Publish:** Gửi dữ liệu từ client lên server (thường qua các endpoint bắt đầu bằng `/app`).
- **Topic:** Kênh chung, nhiều client có thể cùng subscribe (ví dụ: `/topic/chat/1`).
- **Queue:** Kênh riêng tư, chỉ một client nhận (ít dùng cho chat 1-1).

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
  // Xử lý các loại message: MESSAGE, TYPING, READ_RECEIPT
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
> Tin nhắn sẽ được gửi tới tất cả client qua WebSocket sau khi backend xử lý.

---

## 8. Gửi trạng thái "đang nhập" (typing)

```js
client.publish({
  destination: '/app/chat.typing',
  body: JSON.stringify({
    chatRoomId: 1,
    typing: true, // hoặc false khi dừng nhập
  }),
});
```

---

## 9. Đánh dấu tin nhắn đã đọc

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

## 10. Xử lý các loại message từ WebSocket

```js
client.subscribe('/topic/chat/1', (message) => {
  const data = JSON.parse(message.body);
  switch (data.type) {
    case 'MESSAGE':
      // Hiển thị tin nhắn mới
      break;
    case 'TYPING':
      // Hiển thị "đang nhập..." cho user khác
      break;
    case 'READ_RECEIPT':
      // Đánh dấu tin nhắn đã được đọc
      break;
    default:
      // Xử lý các loại khác nếu có
  }
});
```

---

## 11. Đăng ký nhận thông báo matching thành công (Match Notification)

### Đăng ký (subscribe) topic thông báo match

Khi user đăng nhập hoặc vào app, hãy subscribe vào topic thông báo cá nhân của user (ví dụ: `/topic/notification/{userId}`):

```js
// Giả sử userId là 123
client.subscribe('/topic/notification/123', (message) => {
  const data = JSON.parse(message.body);
  if (data.type === 'MATCH') {
    // Xử lý thông báo match thành công
    alert(`Bạn đã match với ${data.matchedUsername}!`);
    // Có thể tự động mở phòng chat hoặc cập nhật UI
  }
  // Có thể xử lý các loại notification khác ở đây
});
```

#### Cấu trúc message thông báo match (ví dụ)

```json
{
  "type": "MATCH",
  "matchId": 5,
  "chatRoomId": 10,
  "matchedUserId": 456,
  "matchedUsername": "jane_smith",
  "matchMessage": "You and jane_smith have matched! Start chatting now!"
}
```

#### Luồng hoạt động

- Khi user A và user B cùng like nhau (match thành công), backend sẽ gửi thông báo qua WebSocket tới cả hai user.
- Client lắng nghe topic `/topic/notification/{userId}` sẽ nhận được thông báo này ngay lập tức.

#### Gợi ý xử lý UI

- Hiển thị popup thông báo match thành công.
- Tự động chuyển sang giao diện chat với user vừa match (dùng `chatRoomId` trong message).
- Cập nhật danh sách match hoặc chat room.

---

## 12. Lưu ý bảo mật

- Luôn truyền JWT token khi kết nối và gọi API.
- Chỉ user thuộc phòng chat hoặc đúng userId mới nhận được tin nhắn/thông báo.

---

## 13. Tham khảo thêm

- [@stomp/stompjs documentation](https://stomp-js.github.io/stomp-websocket/codo/extra/docs-src/Usage.md.html)
- [SockJS documentation](https://github.com/sockjs/sockjs-client)

---

**Nếu cần ví dụ cụ thể cho mobile hoặc framework khác, hãy hỏi lại để được hướng dẫn chi tiết hơn!** 
