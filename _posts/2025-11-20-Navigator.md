---
title: "TÌm hiểu về Navigator API"
date: 2025-11-20 01:17:00  +0700
categories: [navigator, api]
tags: [javascript]
---


# Tìm Hiểu Navigator API

## Giới thiệu

Navigator API là một interface đại diện cho trạng thái và danh tính của user agent (trình duyệt). Nó cho phép các script truy vấn thông tin và đăng ký để thực hiện một số hoạt động nhất định.

Truy cập Navigator object thông qua: `window.navigator` hoặc đơn giản là `navigator`

---

## 1. Navigator.geolocation

### Mục đích
Cung cấp quyền truy cập vào vị trí địa lý của thiết bị, cho phép website hoặc ứng dụng cung cấp kết quả tùy chỉnh dựa trên vị trí của người dùng.

### Phương thức
- `getCurrentPosition(success, error, options)` - Lấy vị trí hiện tại
- `watchPosition(success, error, options)` - Theo dõi vị trí liên tục
- `clearWatch(id)` - Dừng theo dõi vị trí

### Cách dùng
```javascript
// Lấy vị trí hiện tại
navigator.geolocation.getCurrentPosition(
  (position) => {
    const lat = position.coords.latitude;
    const lon = position.coords.longitude;
    const accuracy = position.coords.accuracy;
    console.log(`Lat: ${lat}, Lon: ${lon}, Accuracy: ${accuracy}m`);
  },
  (error) => {
    console.error('Geolocation error:', error);
  }
);

// Theo dõi vị trí liên tục
const watchId = navigator.geolocation.watchPosition(
  (position) => {
    console.log('Position updated:', position.coords);
  }
);

// Dừng theo dõi
navigator.geolocation.clearWatch(watchId);
```

---

## 2. Navigator.mediaDevices

### Mục đích
Cung cấp quyền truy cập đến các thiết bị media như camera và microphone, cho phép tạo ứng dụng video conference, ghi âm, chụp ảnh.

### Phương thức
- `getUserMedia(constraints)` - Yêu cầu quyền truy cập camera/mic
- `enumerateDevices()` - Liệt kê các thiết bị media có sẵn
- `getSupportedConstraints()` - Lấy danh sách constraints được hỗ trợ
- `getDisplayMedia(constraints)` - Chia sẻ màn hình

### Cách dùng
```javascript
// Truy cập camera và microphone
navigator.mediaDevices.getUserMedia({ 
  video: true, 
  audio: true 
})
.then(stream => {
  const videoElement = document.querySelector('video');
  videoElement.srcObject = stream;
})
.catch(error => {
  console.error('Error accessing media devices:', error);
});

// Liệt kê các thiết bị
navigator.mediaDevices.enumerateDevices()
.then(devices => {
  devices.forEach(device => {
    console.log(`${device.kind}: ${device.label}`);
  });
});

// Chia sẻ màn hình
navigator.mediaDevices.getDisplayMedia({ video: true })
.then(stream => {
  // Xử lý stream màn hình
})
.catch(error => {
  console.error('Screen sharing error:', error);
});
```

---

## 3. Navigator.getBattery()

### Mục đích
Cung cấp thông tin về trạng thái pin của hệ thống, bao gồm mức pin, trạng thái sạc, và thời gian còn lại.

### Phương thức
Trả về một Promise với BatteryManager object có các thuộc tính:
- `level` - Mức pin (0-1)
- `charging` - Đang sạc hay không (boolean)
- `chargingTime` - Thời gian để sạc đầy (giây)
- `dischargingTime` - Thời gian sử dụng còn lại (giây)

### Events
- `levelchange`
- `chargingchange`
- `chargingtimechange`
- `dischargingtimechange`

### Cách dùng
```javascript
navigator.getBattery().then(battery => {
  console.log(`Battery level: ${battery.level * 100}%`);
  console.log(`Charging: ${battery.charging}`);
  
  // Lắng nghe sự kiện thay đổi
  battery.addEventListener('levelchange', () => {
    console.log(`Battery level changed: ${battery.level * 100}%`);
  });
  
  battery.addEventListener('chargingchange', () => {
    console.log(`Charging status: ${battery.charging}`);
  });
});
```

---

## 4. Navigator.clipboard

### Mục đích
Cung cấp quyền truy cập đọc và ghi vào clipboard hệ thống, cho phép implement tính năng copy/paste trong ứng dụng web.

### Phương thức
- `writeText(text)` - Ghi text vào clipboard
- `readText()` - Đọc text từ clipboard
- `write(data)` - Ghi dữ liệu (bao gồm ảnh) vào clipboard
- `read()` - Đọc dữ liệu từ clipboard

### Cách dùng
```javascript
// Ghi text vào clipboard
navigator.clipboard.writeText('Hello, clipboard!')
.then(() => {
  console.log('Text copied to clipboard');
})
.catch(error => {
  console.error('Failed to copy text:', error);
});

// Đọc text từ clipboard
navigator.clipboard.readText()
.then(text => {
  console.log('Clipboard content:', text);
})
.catch(error => {
  console.error('Failed to read clipboard:', error);
});

// Ghi ảnh vào clipboard
fetch('https://example.com/image.png')
.then(response => response.blob())
.then(blob => {
  return navigator.clipboard.write([
    new ClipboardItem({ 'image/png': blob })
  ]);
})
.then(() => {
  console.log('Image copied to clipboard');
});
```

**Lưu ý:** Clipboard API chỉ hoạt động trong secure context (HTTPS hoặc localhost) và cần quyền từ Permissions API.

---

## 5. Navigator.serviceWorker

### Mục đích
Cung cấp quyền truy cập để đăng ký, xóa, nâng cấp và giao tiếp với Service Workers, cho phép tạo Progressive Web Apps (PWAs) với chức năng offline.

### Phương thức
- `register(scriptURL, options)` - Đăng ký service worker
- `getRegistration(scope)` - Lấy registration hiện có
- `getRegistrations()` - Lấy tất cả registrations
- `ready` - Promise khi service worker sẵn sàng

### Cách dùng
```javascript
// Kiểm tra hỗ trợ
if ('serviceWorker' in navigator) {
  // Đăng ký service worker
  navigator.serviceWorker.register('/service-worker.js')
  .then(registration => {
    console.log('Service worker registered:', registration);
    
    // Kiểm tra cập nhật
    registration.update();
  })
  .catch(error => {
    console.error('Service worker registration failed:', error);
  });
  
  // Lắng nghe message từ service worker
  navigator.serviceWorker.addEventListener('message', event => {
    console.log('Message from service worker:', event.data);
  });
  
  // Gửi message đến service worker
  navigator.serviceWorker.controller?.postMessage({
    type: 'SYNC_DATA',
    data: { key: 'value' }
  });
}
```

---

## 6. Navigator.share()

### Mục đích
Gọi native sharing mechanism của platform để chia sẻ text, URLs hoặc files đến các ứng dụng khác.

### Phương thức
- `share(data)` - Chia sẻ dữ liệu
- `canShare(data)` - Kiểm tra có thể chia sẻ không

### Cách dùng
```javascript
const shareData = {
  title: 'MDN Web Docs',
  text: 'Learn web development on MDN!',
  url: 'https://developer.mozilla.org'
};

// Kiểm tra có thể chia sẻ không
if (navigator.canShare && navigator.canShare(shareData)) {
  // Thực hiện chia sẻ
  navigator.share(shareData)
  .then(() => {
    console.log('Content shared successfully');
  })
  .catch(error => {
    console.error('Error sharing:', error);
  });
}

// Chia sẻ file
const shareButton = document.querySelector('#share-button');
shareButton.addEventListener('click', async () => {
  const files = document.querySelector('#file-input').files;
  
  if (navigator.canShare && navigator.canShare({ files })) {
    try {
      await navigator.share({
        files: files,
        title: 'My Images',
        text: 'Check out these images!'
      });
      console.log('Files shared successfully');
    } catch (error) {
      console.error('Error sharing files:', error);
    }
  }
});
```

---

## 7. Navigator.permissions

### Mục đích
Cung cấp quyền truy vấn và cập nhật trạng thái permission của các API được bảo vệ.

### Phương thức
- `query(permissionDescriptor)` - Truy vấn trạng thái permission

### Permissions có thể truy vấn
- `geolocation`
- `notifications`
- `push`
- `midi`
- `camera`
- `microphone`
- `clipboard-read`
- `clipboard-write`

### Cách dùng
```javascript
// Kiểm tra permission geolocation
navigator.permissions.query({ name: 'geolocation' })
.then(result => {
  console.log('Geolocation permission:', result.state);
  // result.state: 'granted', 'denied', or 'prompt'
  
  // Lắng nghe thay đổi
  result.addEventListener('change', () => {
    console.log('Permission changed:', result.state);
  });
});

// Kiểm tra nhiều permissions
async function checkPermissions() {
  const geoPermission = await navigator.permissions.query({ 
    name: 'geolocation' 
  });
  
  const notifPermission = await navigator.permissions.query({ 
    name: 'notifications' 
  });
  
  console.log('Geolocation:', geoPermission.state);
  console.log('Notifications:', notifPermission.state);
}

checkPermissions();
```

---

## 8. Navigator.bluetooth

### Mục đích
Cung cấp quyền truy cập Web Bluetooth API để kết nối và tương tác với các thiết bị Bluetooth Low Energy (BLE).

### Phương thức
- `requestDevice(options)` - Yêu cầu kết nối với thiết bị BLE
- `getAvailability()` - Kiểm tra Bluetooth có khả dụng không
- `getDevices()` - Lấy danh sách thiết bị đã kết nối

### Cách dùng
```javascript
// Kiểm tra hỗ trợ Bluetooth
if ('bluetooth' in navigator) {
  // Yêu cầu kết nối với thiết bị có dịch vụ battery
  navigator.bluetooth.requestDevice({
    filters: [{ services: ['battery_service'] }]
  })
  .then(device => {
    console.log('Device Name:', device.name);
    return device.gatt.connect();
  })
  .then(server => {
    console.log('Connected to GATT server');
    return server.getPrimaryService('battery_service');
  })
  .then(service => {
    return service.getCharacteristic('battery_level');
  })
  .then(characteristic => {
    return characteristic.readValue();
  })
  .then(value => {
    const batteryLevel = value.getUint8(0);
    console.log('Battery Level:', batteryLevel + '%');
  })
  .catch(error => {
    console.error('Bluetooth error:', error);
  });
  
  // Kết nối theo tên thiết bị
  navigator.bluetooth.requestDevice({
    filters: [{ name: 'My Device' }],
    optionalServices: ['battery_service']
  })
  .then(device => {
    console.log('Found device:', device.name);
  });
  
  // Kết nối với bất kỳ thiết bị nào
  navigator.bluetooth.requestDevice({
    acceptAllDevices: true,
    optionalServices: ['battery_service']
  })
  .then(device => {
    // Xử lý thiết bị
  });
}
```

**Lưu ý:** Web Bluetooth chỉ hoạt động trong secure context (HTTPS hoặc localhost).

---

## 9. Navigator.credentials

### Mục đích
Cung cấp interface Credentials Management API để yêu cầu credentials và thông báo cho user agent về các sự kiện quan trọng như đăng nhập/đăng xuất thành công.

### Phương thức
- `get(options)` - Yêu cầu credentials từ user
- `store(credential)` - Lưu credentials
- `create(options)` - Tạo credentials mới
- `preventSilentAccess()` - Ngăn đăng nhập tự động

### Cách dùng
```javascript
// Lấy credentials đã lưu
navigator.credentials.get({
  password: true,
  mediation: 'optional'
})
.then(credential => {
  if (credential) {
    console.log('Credential type:', credential.type);
    // Sử dụng credential để đăng nhập
  }
});

// Tạo và lưu password credential
const cred = new PasswordCredential({
  id: 'user@example.com',
  password: 'password123',
  name: 'John Doe'
});

navigator.credentials.store(cred)
.then(() => {
  console.log('Credential stored');
});

// WebAuthn - Tạo public key credential
navigator.credentials.create({
  publicKey: {
    challenge: new Uint8Array([/* challenge data */]),
    rp: {
      name: "Example Corp",
      id: "example.com"
    },
    user: {
      id: new Uint8Array([/* user id */]),
      name: "user@example.com",
      displayName: "John Doe"
    },
    pubKeyCredParams: [
      { type: "public-key", alg: -7 }
    ]
  }
})
.then(credential => {
  console.log('Public key credential created');
});

// Ngăn đăng nhập tự động
navigator.credentials.preventSilentAccess()
.then(() => {
  console.log('Silent access prevented');
});
```

---

## 10. Navigator.storage

### Mục đích
Cung cấp singleton StorageManager object để quản lý persistence permissions và ước tính dung lượng lưu trữ có sẵn.

### Phương thức
- `estimate()` - Ước tính dung lượng đã dùng và khả dụng
- `persist()` - Yêu cầu persistent storage
- `persisted()` - Kiểm tra persistent storage có được cấp không

### Cách dùng
```javascript
// Ước tính dung lượng storage
navigator.storage.estimate()
.then(estimate => {
  console.log('Usage:', estimate.usage, 'bytes');
  console.log('Quota:', estimate.quota, 'bytes');
  console.log('Percentage used:', 
    (estimate.usage / estimate.quota * 100).toFixed(2) + '%');
});

// Yêu cầu persistent storage
navigator.storage.persist()
.then(persistent => {
  if (persistent) {
    console.log('Storage will not be cleared except by explicit user action');
  } else {
    console.log('Storage may be cleared by the UA under storage pressure');
  }
});

// Kiểm tra persistent storage
navigator.storage.persisted()
.then(persistent => {
  console.log('Persistent storage:', persistent);
});
```

---

## 11. Navigator.connection

### Mục đích
Cung cấp thông tin về kết nối mạng của thiết bị thông qua NetworkInformation object.

### Thuộc tính
- `effectiveType` - Loại kết nối hiệu quả ('slow-2g', '2g', '3g', '4g')
- `downlink` - Băng thông tải xuống (Mbps)
- `rtt` - Round-trip time (ms)
- `saveData` - Data saver có bật không

### Events
- `change` - Khi kết nối thay đổi

### Cách dùng
```javascript
if ('connection' in navigator) {
  const connection = navigator.connection;
  
  console.log('Effective type:', connection.effectiveType);
  console.log('Downlink:', connection.downlink, 'Mbps');
  console.log('RTT:', connection.rtt, 'ms');
  console.log('Data saver:', connection.saveData);
  
  // Lắng nghe thay đổi kết nối
  connection.addEventListener('change', () => {
    console.log('Connection changed:', {
      effectiveType: connection.effectiveType,
      downlink: connection.downlink,
      rtt: connection.rtt
    });
    
    // Điều chỉnh chất lượng nội dung dựa trên kết nối
    if (connection.effectiveType === 'slow-2g' || connection.saveData) {
      // Load nội dung chất lượng thấp
      loadLowQualityContent();
    } else {
      // Load nội dung chất lượng cao
      loadHighQualityContent();
    }
  });
}
```

---

## 12. Navigator.onLine

### Mục đích
Trả về boolean cho biết trình duyệt có đang online hay không.

### Cách dùng
```javascript
// Kiểm tra trạng thái online
if (navigator.onLine) {
  console.log('Browser is online');
} else {
  console.log('Browser is offline');
}

// Lắng nghe sự kiện online/offline
window.addEventListener('online', () => {
  console.log('Browser went online');
  syncData(); // Đồng bộ dữ liệu khi online
});

window.addEventListener('offline', () => {
  console.log('Browser went offline');
  showOfflineMessage(); // Hiển thị thông báo offline
});
```

---

## 13. Navigator.vibrate()

### Mục đích
Tạo rung động trên thiết bị hỗ trợ tính năng này (thường là smartphone).

### Cách dùng
```javascript
// Rung 200ms
navigator.vibrate(200);

// Rung với pattern: [rung, dừng, rung, dừng, ...]
navigator.vibrate([100, 50, 100, 50, 100]);

// Dừng rung
navigator.vibrate(0);
// hoặc
navigator.vibrate([]);

// Ví dụ: Rung khi nhấn nút
document.querySelector('#vibrate-button').addEventListener('click', () => {
  if ('vibrate' in navigator) {
    navigator.vibrate([50, 100, 50]);
  }
});
```

---

## 14. Navigator.sendBeacon()

### Mục đích
Gửi dữ liệu nhỏ không đồng bộ qua HTTP đến server, thường dùng để gửi analytics/logging trước khi trang đóng.

### Cách dùng
```javascript
// Gửi dữ liệu analytics khi user rời trang
window.addEventListener('beforeunload', () => {
  const data = JSON.stringify({
    url: window.location.href,
    timestamp: Date.now(),
    userAction: 'page_exit'
  });
  
  navigator.sendBeacon('/analytics', data);
});

// Gửi với FormData
const formData = new FormData();
formData.append('event', 'button_click');
formData.append('buttonId', 'submit-btn');

navigator.sendBeacon('/track', formData);

// Gửi với Blob
const blob = new Blob(
  [JSON.stringify({ event: 'page_view', page: '/home' })],
  { type: 'application/json' }
);

navigator.sendBeacon('/log', blob);
```

---

## 15. Navigator.language và Navigator.languages

### Mục đích
Cung cấp thông tin về ngôn ngữ ưa thích của người dùng.

### Cách dùng
```javascript
// Ngôn ngữ chính
console.log('User language:', navigator.language);
// Ví dụ: "en-US", "vi-VN", "ja-JP"

// Danh sách các ngôn ngữ theo thứ tự ưu tiên
console.log('User languages:', navigator.languages);
// Ví dụ: ["en-US", "en", "vi"]

// Sử dụng để hiển thị nội dung phù hợp
function loadLocalizedContent() {
  const userLang = navigator.language;
  
  if (userLang.startsWith('vi')) {
    loadVietnameseContent();
  } else if (userLang.startsWith('ja')) {
    loadJapaneseContent();
  } else {
    loadEnglishContent();
  }
}
```

---

## 16. Navigator.userAgent

### Mục đích
Trả về user agent string của trình duyệt hiện tại, chứa thông tin về trình duyệt, hệ điều hành, và device.

### Cách dùng
```javascript
const userAgent = navigator.userAgent;
console.log('User Agent:', userAgent);

// Phát hiện trình duyệt (không khuyến khích - nên dùng feature detection)
function getBrowserName(userAgent) {
  if (userAgent.includes("Firefox")) {
    return "Mozilla Firefox";
  } else if (userAgent.includes("Edg")) {
    return "Microsoft Edge";
  } else if (userAgent.includes("Chrome")) {
    return "Google Chrome";
  } else if (userAgent.includes("Safari")) {
    return "Apple Safari";
  } else if (userAgent.includes("Opera") || userAgent.includes("OPR")) {
    return "Opera";
  }
  return "Unknown";
}

console.log('Browser:', getBrowserName(navigator.userAgent));

// Phát hiện mobile
const isMobile = /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i
  .test(navigator.userAgent);
console.log('Is mobile:', isMobile);
```

**Lưu ý:** User Agent string có thể bị giả mạo. Nên sử dụng feature detection thay vì browser detection.

---

## 17. Navigator.cookieEnabled

### Mục đích
Trả về boolean cho biết cookies có được bật trong trình duyệt hay không.

### Cách dùng
```javascript
if (navigator.cookieEnabled) {
  console.log('Cookies are enabled');
  // Có thể sử dụng cookies
  document.cookie = "username=John Doe";
} else {
  console.log('Cookies are disabled');
  // Sử dụng phương pháp lưu trữ khác (localStorage, sessionStorage)
  localStorage.setItem('username', 'John Doe');
}
```

---

## 18. Navigator.hardwareConcurrency

### Mục đích
Trả về số lượng logical processor cores có sẵn trên thiết bị.

### Cách dùng
```javascript
const cores = navigator.hardwareConcurrency;
console.log('Number of CPU cores:', cores);

// Tối ưu hóa số lượng Web Workers dựa trên số cores
const workerCount = Math.max(1, cores - 1); // Giữ 1 core cho UI thread

for (let i = 0; i < workerCount; i++) {
  const worker = new Worker('worker.js');
  worker.postMessage({ task: 'process', data: dataChunks[i] });
}
```

---

## 19. Navigator.deviceMemory

### Mục đích
Trả về dung lượng RAM của thiết bị (tính bằng gigabytes), được làm tròn đến lũy thừa của 2 gần nhất.

### Cách dùng
```javascript
if ('deviceMemory' in navigator) {
  const memory = navigator.deviceMemory;
  console.log('Device memory:', memory, 'GB');
  
  // Điều chỉnh tính năng dựa trên RAM
  if (memory < 2) {
    // Thiết bị RAM thấp - load phiên bản lite
    loadLiteVersion();
  } else if (memory >= 4) {
    // Thiết bị RAM cao - load đầy đủ tính năng
    loadFullFeatures();
  }
}
```

---

## 20. Navigator.maxTouchPoints

### Mục đích
Trả về số lượng điểm chạm đồng thời tối đa được hỗ trợ bởi thiết bị.

### Cách dùng
```javascript
const maxTouchPoints = navigator.maxTouchPoints;
console.log('Max simultaneous touch points:', maxTouchPoints);

// Phát hiện thiết bị cảm ứng
const isTouchDevice = maxTouchPoints > 0;
console.log('Is touch device:', isTouchDevice);

// Điều chỉnh UI cho touch
if (isTouchDevice) {
  // Tăng kích thước button cho dễ touch
  document.body.classList.add('touch-enabled');
}

// Xử lý multi-touch gestures
if (maxTouchPoints >= 2) {
  // Hỗ trợ pinch-to-zoom, two-finger swipe, etc.
  enableMultiTouchGestures();
}
```

---

## 21. Navigator.mediaSession

### Mục đích
Cung cấp MediaSession object để cung cấp metadata và xử lý media controls toàn cục (như trong notification hoặc lock screen).

### Cách dùng
```javascript
if ('mediaSession' in navigator) {
  // Set metadata
  navigator.mediaSession.metadata = new MediaMetadata({
    title: 'Song Title',
    artist: 'Artist Name',
    album: 'Album Name',
    artwork: [
      { src: 'cover-96.png', sizes: '96x96', type: 'image/png' },
      { src: 'cover-128.png', sizes: '128x128', type: 'image/png' },
      { src: 'cover-256.png', sizes: '256x256', type: 'image/png' }
    ]
  });
  
  // Xử lý media controls
  navigator.mediaSession.setActionHandler('play', () => {
    audioElement.play();
  });
  
  navigator.mediaSession.setActionHandler('pause', () => {
    audioElement.pause();
  });
  
  navigator.mediaSession.setActionHandler('seekbackward', () => {
    audioElement.currentTime = Math.max(audioElement.currentTime - 10, 0);
  });
  
  navigator.mediaSession.setActionHandler('seekforward', () => {
    audioElement.currentTime = Math.min(
      audioElement.currentTime + 10,
      audioElement.duration
    );
  });
  
  navigator.mediaSession.setActionHandler('previoustrack', () => {
    playPreviousSong();
  });
  
  navigator.mediaSession.setActionHandler('nexttrack', () => {
    playNextSong();
  });
}
```

---

## 22. Navigator.gpu

### Mục đích
Trả về GPU object cho browsing context hiện tại, là entry point cho WebGPU API để truy cập GPU.

### Cách dùng
```javascript
if ('gpu' in navigator) {
  // Request GPU adapter
  const adapter = await navigator.gpu.requestAdapter();
  
  if (!adapter) {
    console.error('No GPU adapter found');
    return;
  }
  
  // Request GPU device
  const device = await adapter.requestDevice();
  
  console.log('GPU Adapter:', adapter);
  console.log('GPU Device:', device);
  
  // Sử dụng WebGPU cho rendering/computation
  const canvas = document.querySelector('canvas');
  const context = canvas.getContext('webgpu');
  
  const format = navigator.gpu.getPreferredCanvasFormat();
  context.configure({
    device: device,
    format: format
  });
  
  // Thực hiện GPU operations...
}
```

---

## 23. Navigator.keyboard

### Mục đích
Trả về Keyboard object để truy cập các chức năng như lấy keyboard layout maps và toggle capturing key presses.

### Phương thức
- `getLayoutMap()` - Lấy keyboard layout map
- `lock(keyCodes)` - Lock phím để ứng dụng nhận
- `unlock()` - Unlock phím

### Cách dùng
```javascript
if ('keyboard' in navigator) {
  // Lấy keyboard layout
  navigator.keyboard.getLayoutMap()
  .then(layoutMap => {
    console.log('Keyboard layout:', layoutMap);
    
    // Kiểm tra key code
    const keyW = layoutMap.get('KeyW');
    console.log('Key W:', keyW);
  });
  
  // Lock phím để game nhận đầy đủ input
  const gameKeys = ['KeyW', 'KeyA', 'KeyS', 'KeyD', 'Space'];
  
  document.querySelector('#fullscreen-btn').addEventListener('click', async () => {
    await document.body.requestFullscreen();
    await navigator.keyboard.lock(gameKeys);
    console.log('Keyboard locked for game controls');
  });
  
  // Unlock khi thoát fullscreen
  document.addEventListener('fullscreenchange', () => {
    if (!document.fullscreenElement) {
      navigator.keyboard.unlock();
      console.log('Keyboard unlocked');
    }
  });
}
```

---

## 24. Navigator.locks

### Mục đích
Trả về LockManager object để quản lý locks, ngăn race conditions khi truy cập shared resources.

### Phương thức
- `request(name, options, callback)` - Yêu cầu lock
- `query()` - Truy vấn locks hiện tại

### Cách dùng
```javascript
if ('locks' in navigator) {
  // Request exclusive lock
  await navigator.locks.request('my_resource', async lock => {
    // Code này chỉ chạy khi có được lock
    console.log('Lock acquired');
    
    // Truy cập shared resource
    await updateSharedResource();
    
    // Lock tự động release khi function hoàn thành
  });
  
  // Request shared lock (multiple readers)
  await navigator.locks.request('my_resource', 
    { mode: 'shared' },
    async lock => {
      // Nhiều shared locks có thể tồn tại cùng lúc
      await readSharedResource();
    }
  );
  
  // Request lock với timeout
  try {
    await navigator.locks.request('my_resource',
      { 
        mode: 'exclusive',
        ifAvailable: true  // Fail nếu không có sẵn ngay
      },
      async lock => {
        if (!lock) {
          console.log('Lock not available');
          return;
        }
        await updateSharedResource();
      }
    );
  } catch (error) {
    console.error('Lock request failed:', error);
  }
  
  // Query hiện trạng locks
  const state = await navigator.locks.query();
  console.log('Held locks:', state.held);
  console.log('Pending locks:', state.pending);
}
```

---

## 25. Navigator.presentation

### Mục đích
Trả về Presentation API để khởi tạo và điều khiển presentations trên màn hình thứ hai.

### Cách dùng
```javascript
if ('presentation' in navigator) {
  // Request presentation display
  const presentationRequest = new PresentationRequest([
    'https://example.com/presentation.html'
  ]);
  
  // Start presentation
  document.querySelector('#present-btn').addEventListener('click', () => {
    presentationRequest.start()
    .then(connection => {
      console.log('Presentation started:', connection);
      
      // Gửi message đến presentation
      connection.send(JSON.stringify({
        type: 'slide',
        number: 1
      }));
      
      // Nhận message từ presentation
      connection.addEventListener('message', event => {
        console.log('Message from presentation:', event.data);
      });
      
      // Xử lý khi presentation đóng
      connection.addEventListener('close', () => {
        console.log('Presentation closed');
      });
    })
    .catch(error => {
      console.error('Presentation error:', error);
    });
  });
  
  // Reconnect to existing presentation
  presentationRequest.reconnect('presentation-id')
  .then(connection => {
    console.log('Reconnected to presentation');
  });
  
  // Monitor display availability
  presentationRequest.getAvailability()
  .then(availability => {
    console.log('Display available:', availability.value);
    
    availability.addEventListener('change', () => {
      console.log('Display availability changed:', availability.value);
    });
  });
}
```

---

## 26. Navigator.wakeLock (Screen Wake Lock API)

### Mục đích
Ngăn màn hình tắt hoặc mờ đi, hữu ích cho ứng dụng đọc sách, xem video, navigation.

### Cách dùng
```javascript
if ('wakeLock' in navigator) {
  let wakeLock = null;
  
  // Request wake lock
  async function requestWakeLock() {
    try {
      wakeLock = await navigator.wakeLock.request('screen');
      console.log('Wake Lock is active');
      
      // Lắng nghe khi wake lock bị release
      wakeLock.addEventListener('release', () => {
        console.log('Wake Lock has been released');
      });
    } catch (error) {
      console.error('Wake Lock error:', error);
    }
  }
  
  // Release wake lock
  async function releaseWakeLock() {
    if (wakeLock !== null) {
      await wakeLock.release();
      wakeLock = null;
      console.log('Wake Lock released manually');
    }
  }
  
  // Tự động request lại khi visibility thay đổi
  document.addEventListener('visibilitychange', async () => {
    if (document.visibilityState === 'visible' && wakeLock === null) {
      await requestWakeLock();
    }
  });
  
  // Sử dụng
  document.querySelector('#start-btn').addEventListener('click', requestWakeLock);
  document.querySelector('#stop-btn').addEventListener('click', releaseWakeLock);
}
```

---

## 27. Navigator.userActivation

### Mục đích
Cung cấp thông tin về user activation state của window hiện tại.

### Thuộc tính
- `hasBeenActive` - User có tương tác với trang chưa
- `isActive` - Có transient activation hiện tại không

### Cách dùng
```javascript
if ('userActivation' in navigator) {
  console.log('Has been active:', navigator.userActivation.hasBeenActive);
  console.log('Is active:', navigator.userActivation.isActive);
  
  // Một số API yêu cầu user activation
  document.querySelector('#play-btn').addEventListener('click', async () => {
    if (navigator.userActivation.isActive) {
      // Có user activation - có thể autoplay
      await videoElement.play();
    } else {
      console.log('No user activation - cannot autoplay');
    }
  });
}
```

---

## 28. Navigator.scheduling

### Mục đích
Cung cấp Scheduler API để schedule tasks với priority khác nhau.

### Cách dùng
```javascript
if ('scheduling' in navigator && 'scheduler' in navigator.scheduling) {
  const scheduler = navigator.scheduling.scheduler;
  
  // Post task với priority
  scheduler.postTask(
    () => {
      // High priority task
      console.log('High priority task executed');
      performCriticalUpdate();
    },
    { priority: 'user-blocking' }
  );
  
  scheduler.postTask(
    () => {
      // Normal priority task
      console.log('Normal priority task executed');
      updateUI();
    },
    { priority: 'user-visible' }
  );
  
  scheduler.postTask(
    () => {
      // Low priority task
      console.log('Low priority task executed');
      analyzeData();
    },
    { priority: 'background' }
  );
}
```

---

## 29. Navigator.contacts (Contact Picker API)

### Mục đích
Cho phép user chọn contacts từ danh bạ và chia sẻ thông tin hạn chế với website.

### Cách dùng
```javascript
if ('contacts' in navigator) {
  // Kiểm tra properties có sẵn
  const supported = await navigator.contacts.getProperties();
  console.log('Supported properties:', supported);
  
  // Request contacts
  document.querySelector('#select-contacts').addEventListener('click', async () => {
    try {
      const contacts = await navigator.contacts.select(
        ['name', 'email', 'tel'],
        { multiple: true }
      );
      
      contacts.forEach(contact => {
        console.log('Name:', contact.name);
        console.log('Email:', contact.email);
        console.log('Tel:', contact.tel);
      });
    } catch (error) {
      console.error('Contact selection error:', error);
    }
  });
}
```

---

## 30. Navigator.hid (WebHID API)

### Mục đích
Cung cấp quyền truy cập để kết nối với HID devices (Human Interface Devices) như game controllers, keyboards đặc biệt.

### Phương thức
- `requestDevice(options)` - Request HID device
- `getDevices()` - Lấy devices đã granted

### Cách dùng
```javascript
if ('hid' in navigator) {
  // Request HID device
  document.querySelector('#connect-btn').addEventListener('click', async () => {
    try {
      const devices = await navigator.hid.requestDevice({
        filters: [
          { vendorId: 0x1234 } // Filter theo vendor ID
        ]
      });
      
      if (devices.length > 0) {
        const device = devices[0];
        console.log('Device:', device);
        
        // Open device
        await device.open();
        console.log('Device opened');
        
        // Listen for input reports
        device.addEventListener('inputreport', event => {
          const { data, device, reportId } = event;
          console.log('Input report:', {
            reportId,
            data: new Uint8Array(data.buffer)
          });
        });
        
        // Send output report
        const reportId = 0x01;
        const data = new Uint8Array([0x00, 0x01, 0x02]);
        await device.sendReport(reportId, data);
      }
    } catch (error) {
      console.error('HID error:', error);
    }
  });
  
  // Get previously granted devices
  const devices = await navigator.hid.getDevices();
  console.log('Granted HID devices:', devices);
  
  // Listen for device connection
  navigator.hid.addEventListener('connect', event => {
    console.log('HID device connected:', event.device);
  });
  
  navigator.hid.addEventListener('disconnect', event => {
    console.log('HID device disconnected:', event.device);
  });
}
```

---

## 31. Navigator.usb (WebUSB API)

### Mục đích
Cung cấp quyền truy cập WebUSB API để điều khiển USB devices.

### Phương thức
- `requestDevice(options)` - Request USB device
- `getDevices()` - Lấy devices đã granted

### Cách dùng
```javascript
if ('usb' in navigator) {
  // Request USB device
  document.querySelector('#connect-usb').addEventListener('click', async () => {
    try {
      const device = await navigator.usb.requestDevice({
        filters: [
          { vendorId: 0x2341 } // Arduino vendor ID
        ]
      });
      
      console.log('USB Device:', device);
      
      // Open device
      await device.open();
      await device.selectConfiguration(1);
      await device.claimInterface(0);
      
      // Transfer data
      const result = await device.transferOut(1, new Uint8Array([0x01, 0x02]));
      console.log('Transfer result:', result);
      
      // Read data
      const data = await device.transferIn(1, 64);
      console.log('Received data:', new Uint8Array(data.data.buffer));
      
      // Close device
      await device.close();
    } catch (error) {
      console.error('USB error:', error);
    }
  });
  
  // Listen for device connection
  navigator.usb.addEventListener('connect', event => {
    console.log('USB device connected:', event.device);
  });
  
  navigator.usb.addEventListener('disconnect', event => {
    console.log('USB device disconnected:', event.device);
  });
}
```

---

## 32. Navigator.serial (Web Serial API)

### Mục đích
Cung cấp quyền truy cập Web Serial API để điều khiển serial ports.

### Phương thức
- `requestPort(options)` - Request serial port
- `getPorts()` - Lấy ports đã granted

### Cách dùng
```javascript
if ('serial' in navigator) {
  // Request serial port
  document.querySelector('#connect-serial').addEventListener('click', async () => {
    try {
      const port = await navigator.serial.requestPort();
      console.log('Serial port:', port);
      
      // Open port với config
      await port.open({ 
        baudRate: 9600,
        dataBits: 8,
        stopBits: 1,
        parity: 'none'
      });
      
      // Write data
      const writer = port.writable.getWriter();
      const data = new Uint8Array([0x48, 0x65, 0x6C, 0x6C, 0x6F]); // "Hello"
      await writer.write(data);
      writer.releaseLock();
      
      // Read data
      const reader = port.readable.getReader();
      while (true) {
        const { value, done } = await reader.read();
        if (done) break;
        console.log('Received:', new TextDecoder().decode(value));
      }
      reader.releaseLock();
      
      // Close port
      await port.close();
    } catch (error) {
      console.error('Serial error:', error);
    }
  });
  
  // Get previously granted ports
  const ports = await navigator.serial.getPorts();
  console.log('Granted serial ports:', ports);
}
```

---

## 33. Navigator.xr (WebXR Device API)

### Mục đích
Trả về XRSystem object, là entry point cho WebXR API để tạo trải nghiệm VR/AR.

### Cách dùng
```javascript
if ('xr' in navigator) {
  // Kiểm tra hỗ trợ VR
  navigator.xr.isSessionSupported('immersive-vr')
  .then(supported => {
    if (supported) {
      console.log('VR is supported');
      document.querySelector('#enter-vr').style.display = 'block';
    }
  });
  
  // Kiểm tra hỗ trợ AR
  navigator.xr.isSessionSupported('immersive-ar')
  .then(supported => {
    if (supported) {
      console.log('AR is supported');
      document.querySelector('#enter-ar').style.display = 'block';
    }
  });
  
  // Request VR session
  document.querySelector('#enter-vr').addEventListener('click', async () => {
    try {
      const session = await navigator.xr.requestSession('immersive-vr', {
        requiredFeatures: ['local-floor']
      });
      
      console.log('VR session started');
      
      // Setup WebGL context
      const canvas = document.createElement('canvas');
      const gl = canvas.getContext('webgl', { xrCompatible: true });
      
      // Setup session
      await session.updateRenderState({
        baseLayer: new XRWebGLLayer(session, gl)
      });
      
      // Request animation frame
      const referenceSpace = await session.requestReferenceSpace('local-floor');
      
      function onXRFrame(time, frame) {
        session.requestAnimationFrame(onXRFrame);
        
        const pose = frame.getViewerPose(referenceSpace);
        if (pose) {
          // Render VR scene
          renderVRScene(pose);
        }
      }
      
      session.requestAnimationFrame(onXRFrame);
      
      // Handle session end
      session.addEventListener('end', () => {
        console.log('VR session ended');
      });
    } catch (error) {
      console.error('XR error:', error);
    }
  });
}
```

---

## 34. Navigator.requestMIDIAccess()

### Mục đích
Yêu cầu quyền truy cập MIDI devices trên hệ thống của user.

### Cách dùng
```javascript
// Request MIDI access
navigator.requestMIDIAccess()
.then(midiAccess => {
  console.log('MIDI access granted');
  
  // List MIDI inputs
  const inputs = midiAccess.inputs.values();
  for (let input of inputs) {
    console.log('MIDI Input:', input.name);
    
    // Listen for MIDI messages
    input.onmidimessage = event => {
      const [command, note, velocity] = event.data;
      console.log('MIDI message:', { command, note, velocity });
      
      // Example: handle note on/off
      if (command === 144) { // Note on
        playNote(note, velocity);
      } else if (command === 128) { // Note off
        stopNote(note);
      }
    };
  }
  
  // List MIDI outputs
  const outputs = midiAccess.outputs.values();
  for (let output of outputs) {
    console.log('MIDI Output:', output.name);
    
    // Send MIDI message
    // Note on: channel 1, note 60 (middle C), velocity 100
    output.send([144, 60, 100]);
    
    // Note off after 1 second
    setTimeout(() => {
      output.send([128, 60, 0]);
    }, 1000);
  }
  
  // Listen for device connection changes
  midiAccess.onstatechange = event => {
    console.log('MIDI device state changed:', event.port);
  };
})
.catch(error => {
  console.error('MIDI access error:', error);
});
```

---

## 35. Navigator.requestMediaKeySystemAccess()

### Mục đích
Yêu cầu quyền truy cập Media Key System cho DRM (Digital Rights Management).

### Cách dùng
```javascript
// Request media key system access
const keySystemConfig = [{
  initDataTypes: ['cenc'],
  audioCapabilities: [{
    contentType: 'audio/mp4; codecs="mp4a.40.2"'
  }],
  videoCapabilities: [{
    contentType: 'video/mp4; codecs="avc1.42E01E"'
  }]
}];

navigator.requestMediaKeySystemAccess('com.widevine.alpha', keySystemConfig)
.then(keySystemAccess => {
  console.log('Media key system access granted');
  return keySystemAccess.createMediaKeys();
})
.then(mediaKeys => {
  console.log('Media keys created');
  
  // Set media keys on video element
  const video = document.querySelector('video');
  return video.setMediaKeys(mediaKeys);
})
.then(() => {
  console.log('Media keys set on video element');
  // Load encrypted content
})
.catch(error => {
  console.error('Media key system error:', error);
});
```

---

## 36. Navigator.registerProtocolHandler()

### Mục đích
Đăng ký website như một protocol handler để xử lý các protocol schemes cụ thể.

### Cách dùng
```javascript
// Đăng ký protocol handler
try {
  navigator.registerProtocolHandler(
    'web+coffee',
    'https://example.com/coffee?type=%s',
    'Coffee Protocol Handler'
  );
  console.log('Protocol handler registered');
} catch (error) {
  console.error('Protocol handler error:', error);
}

// Khi user click link: web+coffee:latte
// Browser sẽ mở: https://example.com/coffee?type=latte

// Ví dụ khác: Email handler
navigator.registerProtocolHandler(
  'mailto',
  'https://example.com/mail?to=%s',
  'Example Mail'
);

// RSS feed handler
navigator.registerProtocolHandler(
  'web+feed',
  'https://example.com/feed?url=%s',
  'Feed Reader'
);
```

---

## 37. Navigator.setAppBadge() và clearAppBadge()

### Mục đích
Đặt hoặc xóa badge trên icon của ứng dụng (PWA).

### Cách dùng
```javascript
// Set badge với số
navigator.setAppBadge(5)
.then(() => {
  console.log('App badge set to 5');
})
.catch(error => {
  console.error('Set badge error:', error);
});

// Set badge không có số (hiển thị dot)
navigator.setAppBadge()
.then(() => {
  console.log('App badge set (dot)');
});

// Clear badge
navigator.clearAppBadge()
.then(() => {
  console.log('App badge cleared');
})
.catch(error => {
  console.error('Clear badge error:', error);
});

// Ví dụ: Update badge với số tin nhắn chưa đọc
async function updateUnreadCount(count) {
  if ('setAppBadge' in navigator) {
    if (count > 0) {
      await navigator.setAppBadge(count);
    } else {
      await navigator.clearAppBadge();
    }
  }
}

// Sử dụng
updateUnreadCount(10); // Hiển thị badge "10"
```

---

## 38. Navigator.canShare()

### Mục đích
Kiểm tra xem có thể chia sẻ dữ liệu cụ thể hay không trước khi gọi `share()`.

### Cách dùng
```javascript
const shareData = {
  title: 'Example Title',
  text: 'Example text',
  url: 'https://example.com'
};

// Kiểm tra có thể share không
if (navigator.canShare && navigator.canShare(shareData)) {
  console.log('Can share this data');
  
  // Share
  navigator.share(shareData)
  .then(() => console.log('Shared successfully'))
  .catch(error => console.error('Share error:', error));
} else {
  console.log('Cannot share this data');
  // Fallback: Show copy link button
}

// Kiểm tra có thể share files không
const files = [new File(['content'], 'file.txt', { type: 'text/plain' })];

if (navigator.canShare && navigator.canShare({ files })) {
  console.log('Can share files');
} else {
  console.log('Cannot share files on this platform');
}
```

---

## Bảng Tổng Hợp Browser Support

| API | Chrome | Firefox | Safari | Edge |
|-----|--------|---------|--------|------|
| geolocation | ✅ | ✅ | ✅ | ✅ |
| mediaDevices | ✅ | ✅ | ✅ | ✅ |
| getBattery() | ✅ | ❌ | ❌ | ✅ |
| clipboard | ✅ | ✅ | ✅ | ✅ |
| serviceWorker | ✅ | ✅ | ✅ | ✅ |
| share() | ✅ | ❌ | ✅ | ✅ |
| permissions | ✅ | ✅ | ⚠️ | ✅ |
| bluetooth | ✅ | ❌ | ❌ | ✅ |
| credentials | ✅ | ✅ | ✅ | ✅ |
| storage | ✅ | ✅ | ✅ | ✅ |
| connection | ✅ | ❌ | ❌ | ✅ |
| vibrate() | ✅ | ✅ | ❌ | ✅ |
| sendBeacon() | ✅ | ✅ | ✅ | ✅ |
| wakeLock | ✅ | ✅ | ✅ | ✅ |
| contacts | ✅ | ❌ | ❌ | ✅ |
| hid | ✅ | ❌ | ❌ | ✅ |
| usb | ✅ | ❌ | ❌ | ✅ |
| serial | ✅ | ❌ | ❌ | ✅ |
| xr | ✅ | ❌ | ❌ | ✅ |

**Ghi chú:**
- ✅ = Được hỗ trợ đầy đủ
- ⚠️ = Hỗ trợ một phần
- ❌ = Không được hỗ trợ

---

## Lưu Ý Bảo Mật

1. **Secure Context Required**: Nhiều API chỉ hoạt động trong secure context (HTTPS hoặc localhost)
2. **Permissions**: Hầu hết APIs yêu cầu quyền từ user trước khi sử dụng
3. **User Gesture**: Một số APIs chỉ có thể gọi sau user interaction (click, touch)
4. **Privacy**: Luôn tôn trọng privacy của user và chỉ yêu cầu quyền khi cần thiết

---

## Best Practices

1. **Feature Detection**: Luôn kiểm tra API có tồn tại trước khi sử dụng
```javascript
if ('geolocation' in navigator) {
  // Sử dụng geolocation
}
```

2. **Error Handling**: Luôn xử lý errors và rejections
```javascript
navigator.geolocation.getCurrentPosition(
  success,
  error  // Xử lý error
);
```

3. **Graceful Degradation**: Cung cấp fallback cho browsers không hỗ trợ
```javascript
if ('share' in navigator) {
  await navigator.share(data);
} else {
  // Fallback: Show copy link button
  copyToClipboard(data.url);
}
```

4. **User Consent**: Giải thích rõ tại sao cần quyền trước khi yêu cầu
5. **Progressive Enhancement**: Bắt đầu với chức năng cơ bản, thêm tính năng nâng cao nếu được hỗ trợ

---

## Tài Liệu Tham Khảo

- [MDN Web Docs - Navigator](https://developer.mozilla.org/en-US/docs/Web/API/Navigator)
- [W3C Web APIs](https://www.w3.org/TR/)
- [Can I Use](https://caniuse.com/) - Kiểm tra browser support
- [Web.dev](https://web.dev/) - Best practices và guides

---

## Kết Luận

Navigator API cung cấp một bộ công cụ mạnh mẽ để tương tác với hardware và capabilities của thiết bị. Từ việc truy cập vị trí, camera, Bluetooth đến quản lý permissions và service workers, các APIs này giúp tạo ra các web applications phong phú và tương tác cao.

Khi sử dụng các APIs này, hãy luôn:
- Kiểm tra browser support
- Xử lý errors đúng cách
- Tôn trọng privacy của user
- Cung cấp fallbacks khi cần thiết
- Test trên nhiều devices và browsers

