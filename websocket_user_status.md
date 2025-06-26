
# HƯỚNG DẪN KẾT NỐI WEBSOCKET ĐỂ BIẾT NGƯỜI DÙNG ONLINE/OFFLINE

## 1. Cơ chế hoạt động tổng quát

Luồng hoạt động:
- **Khi user kết nối WebSocket** (vào app/chat):
  - Server xác thực JWT, lấy userId từ token.
  - Đánh dấu user này là online (lưu vào memory).
  - Lấy danh sách các phòng chat mà user này tham gia.
  - Gửi thông báo ONLINE tới các topic `/topic/chat/{chatRoomId}/user-status` mà user này thuộc về.
- **Khi user disconnect WebSocket** (đóng app/tab):
  - Server đánh dấu user này là offline.
  - Gửi thông báo OFFLINE tới các topic `/topic/chat/{chatRoomId}/user-status`.
- **Client:**
  - Khi vào phòng chat, subscribe vào `/topic/chat/{chatRoomId}/user-status`.
  - Khi nhận được message `{ userId, status }`, cập nhật UI trạng thái online/offline của thành viên trong phòng.
  - Khi vào phòng chat, client nên gọi HTTP API để lấy trạng thái online ban đầu của các thành viên.

### Lắng nghe trạng thái online/offline trên client

**Cấu trúc message nhận được:**
```json
{
  "userId": 123,
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
  console.log('✅ WebSocket connected - User status tracking enabled');
  
  // Lắng nghe trạng thái online/offline của các thành viên trong phòng chat 1
  client.subscribe('/topic/chat/1/user-status', (message) => {
    const data = JSON.parse(message.body);
    // data = { userId: 123, status: "ONLINE" | "OFFLINE" }
    updateUserStatusInRoom(data.userId, data.status);
  });
  
  // Lắng nghe trạng thái cho nhiều phòng chat khác
  client.subscribe('/topic/chat/2/user-status', (message) => {
    const data = JSON.parse(message.body);
    updateUserStatusInRoom(data.userId, data.status);
  });
};

client.activate();
```

## 3. Kiểm tra trạng thái online ban đầu qua HTTP API

```js
// Kiểm tra trạng thái online của một user cụ thể
async function checkUserOnlineStatus(userId) {
  try {
    const response = await fetch(`http://localhost:8080/api/users/${userId}/online`, {
      headers: { 
        'Authorization': `Bearer ${jwtToken}` 
      }
    });
    
    if (response.ok) {
      const isOnline = await response.json();
      // isOnline: true/false
      updateUserStatusInRoom(userId, isOnline ? 'ONLINE' : 'OFFLINE');
    }
  } catch (error) {
    console.error('Error checking user online status:', error);
  }
}

// Sử dụng khi vào phòng chat
function onEnterChatRoom(chatRoomId, memberUserIds) {
  // Subscribe để nhận thông báo realtime
  client.subscribe(`/topic/chat/${chatRoomId}/user-status`, (message) => {
    const data = JSON.parse(message.body);
    updateUserStatusInRoom(data.userId, data.status);
  });
  
  // Kiểm tra trạng thái ban đầu của tất cả thành viên
  memberUserIds.forEach(userId => {
    checkUserOnlineStatus(userId);
  });
}
```

## 4. Xử lý UI cho trạng thái online/offline

```js
function updateUserStatusInRoom(userId, status) {
  const userElement = document.querySelector(`[data-user-id="${userId}"]`);
  if (userElement) {
    const statusIndicator = userElement.querySelector('.status-indicator');
    
    if (status === 'ONLINE') {
      statusIndicator.classList.add('online');
      statusIndicator.classList.remove('offline');
      statusIndicator.textContent = '🟢 Online';
    } else {
      statusIndicator.classList.add('offline');
      statusIndicator.classList.remove('online');
      statusIndicator.textContent = '⚫ Offline';
    }
  }
}

// CSS example
const styles = `
  .status-indicator {
    font-size: 12px;
    padding: 2px 6px;
    border-radius: 10px;
    display: inline-block;
  }
  
  .status-indicator.online {
    background-color: #4CAF50;
    color: white;
  }
  
  .status-indicator.offline {
    background-color: #9E9E9E;
    color: white;
  }
`;
```

## 5. Lưu ý quan trọng

### Tự động reconnect
```js
client.onWebSocketClose = () => {
  console.log('WebSocket connection closed - attempting to reconnect...');
  // Client sẽ tự động reconnect sau 5 giây
};

client.onStompError = (frame) => {
  console.error('STOMP error:', frame);
  // Xử lý lỗi STOMP
};
```

### Xử lý khi user logout
```js
function logout() {
  // Disconnect WebSocket trước khi logout
  if (client.connected) {
    client.deactivate();
  }
  
  // Clear token và redirect
  localStorage.removeItem('token');
  window.location.href = '/login';
}
```

### Performance optimization
```js
// Cache trạng thái để tránh gọi API quá nhiều
const userStatusCache = new Map();

async function checkUserOnlineStatus(userId) {
  // Kiểm tra cache trước
  if (userStatusCache.has(userId)) {
    const cachedStatus = userStatusCache.get(userId);
    const timeDiff = Date.now() - cachedStatus.timestamp;
    
    // Cache trong 30 giây
    if (timeDiff < 30000) {
      updateUserStatusInRoom(userId, cachedStatus.status);
      return;
    }
  }
  
  // Gọi API nếu cache expired
  try {
    const response = await fetch(`http://localhost:8080/api/users/${userId}/online`, {
      headers: { 'Authorization': `Bearer ${jwtToken}` }
    });
    
    if (response.ok) {
      const isOnline = await response.json();
      const status = isOnline ? 'ONLINE' : 'OFFLINE';
      
      // Update cache
      userStatusCache.set(userId, {
        status: status,
        timestamp: Date.now()
      });
      
      updateUserStatusInRoom(userId, status);
    }
  } catch (error) {
    console.error('Error checking user online status:', error);
  }
}
```

## 6. API Endpoints

- `GET /api/users/{userId}/online` - Kiểm tra trạng thái online của user
- WebSocket topic: `/topic/chat/{chatRoomId}/user-status` - Nhận thông báo thay đổi trạng thái

## 7. Lưu ý bảo mật

- Chỉ user thuộc phòng chat mới nhận được thông báo trạng thái của các thành viên trong phòng đó
- JWT token được xác thực khi kết nối WebSocket
- Trạng thái online được lưu trong memory (sẽ reset khi restart server)

---

**Tài liệu này đã được cập nhật với thông tin chính xác từ implementation backend và bao gồm các best practices cho việc xử lý trạng thái online/offline.**
