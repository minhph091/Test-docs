# HƯỚNG DẪN KẾT NỐI WEBSOCKET ĐỂ BIẾT NGƯỜI DÙNG ONLINE/OFFLINE

## 1. Cơ chế hoạt động tổng quát

Luồng hoạt động:
- Khi user kết nối WebSocket (vào app/chat):
  - Server xác thực JWT, lấy userId từ token.
  - Đánh dấu user này là online (lưu vào memory hoặc hoặc Redis).
  - Lấy danh sách các phòng chat mà mà user này tham gia.
  - Gửi thông báo ONLINE tới các topic `/topic/chat/{chatRoomId}/user-status` mà user này thuộc về.
- Khi user disconnect WebSocket (đóng app/tab):
  - Server đánh dấu user này là offline.
  - Gửi thông báo OFFLINE tới các topic `/topic/chat/{chatRoomId}/user-status`.
- Client:
  - Khi vào phòng chat, subscribe vào `/topic/chat/{chatRoomId}/user-status`.
  - Khi nhận được message `{ userId, status }`, cập nhật UI trạng thái online/offline của thành viên trong phòng.
  - Khi vào phòng chat, client nên gọi HTTP API để lấy trạng thái online ban đầu của các thành viên.

### Lắng nghe trạng thái online/offline trên client

**Cấu trúc message nhận được:**
```json
{
  "userId": "123",
  "status": "ONLINE" // hoặc "OFFLINE"
}
```

## 2. Cách kết nối WebSocket và lắng nghe trạng thái online/offline

Ví dụ với STOMP JS:

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
  // Lắng nghe trạng thái online/offline của các thành viên trong phòng chat 1
  client.subscribe('/topic/chat/1/user-status', (message) => {
    const data = JSON.parse(message.body);
    // data = { userId: 123, status: "ONLINE" | "OFFLINE" }
    updateUserStatusInRoom(data.userId, data.status);
  });
};

client.activate();
```

Kiểm tra trạng thái online ban đầu qua HTTP API:

```js
fetch('http://localhost:8080/api/users/123/statuses/online', {
  headers: { Authorization: `Bearer ${jwtToken}` }
})

.then(res => res.json())
  .then(isOnline => {
    // isOnline: true/false
    updateUserStatusInRoom(123, isOnline ? 'ONLINE' : 'OFFLINE');
  });
```
