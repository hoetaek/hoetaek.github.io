---
title: 무제 파일
publish: false
tags:
---
# 1. 환경 설정

## 1.1 필수 패키지 설치 및 설정

### Laravel Reverb 설치
```bash
# Laravel Reverb 설치
composer require laravel/reverb

# Reverb 설치 및 기본 설정
php artisan install:broadcasting
```

### 환경 설정
```env
# .env 파일
# 브로드캐스트 드라이버 설정
BROADCAST_DRIVER=reverb

# Laravel Reverb 설정
REVERB_APP_ID=your-app-id
REVERB_APP_KEY=your-app-key
REVERB_APP_SECRET=your-app-secret
REVERB_HOST=127.0.0.1
REVERB_PORT=8080
REVERB_SCHEME=http

# Vue.js에서 사용할 환경변수 (Vite)
VITE_REVERB_HOST="${REVERB_HOST}"
VITE_REVERB_PORT="${REVERB_PORT}"
VITE_REVERB_KEY="${REVERB_APP_KEY}"
```

### Vite 설정
```javascript
// vite.config.js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import vue from '@vitejs/plugin-vue';

export default defineConfig({
    plugins: [
        laravel({
            input: 'resources/js/app.js',
            refresh: true,
        }),
        vue({
            template: {
                transformAssetUrls: {
                    base: null,
                    includeAbsolute: false,
                }
            }
        })
    ],
});
```

# 2. 실시간 통신 구현 예시

## 2.1 Regular Polling

```vue
<!-- resources/js/components/DataPolling.vue -->
<template>
  <div class="polling-component">
    <div v-for="item in items" :key="item.id" class="item">
      {{ item.name }}
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'
import axios from 'axios'

const items = ref([])
let pollingInterval = null

const fetchData = async () => {
  try {
    const response = await axios.get('/api/items')
    items.value = response.data
  } catch (error) {
    console.error('폴링 에러:', error)
  }
}

// 컴포넌트 마운트 시 폴링 시작
onMounted(() => {
  fetchData() // 초기 데이터 로드
  pollingInterval = setInterval(fetchData, 3000)
})

// 컴포넌트 언마운트 시 폴링 중지
onBeforeUnmount(() => {
  if (pollingInterval) clearInterval(pollingInterval)
})
</script>
```

## 2.2 Long Polling

```vue
<!-- resources/js/components/LongPolling.vue -->
<template>
  <div class="long-polling-component">
    <div v-for="update in updates" :key="update.id" class="update">
      {{ update.content }}
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'
import axios from 'axios'

const updates = ref([])
const lastUpdateId = ref(0)
const isPolling = ref(true)

const pollUpdates = async () => {
  if (!isPolling.value) return

  try {
    const response = await axios.get(`/api/updates/${lastUpdateId.value}`)
    const newUpdates = response.data

    if (newUpdates.length > 0) {
      updates.value.push(...newUpdates)
      lastUpdateId.value = newUpdates[newUpdates.length - 1].id
    }

    // 즉시 다음 요청 시작
    pollUpdates()
  } catch (error) {
    console.error('Long polling 에러:', error)
    // 에러 발생 시 잠시 대기 후 재시도
    setTimeout(pollUpdates, 5000)
  }
}

onMounted(() => {
  pollUpdates()
})

onBeforeUnmount(() => {
  isPolling.value = false
})
</script>
```

## 2.3 WebSocket (Laravel Reverb)

```vue
<!-- resources/js/components/RealtimeChat.vue -->
<template>
  <div class="chat-container">
    <div class="messages" ref="messagesContainer">
      <div v-for="message in messages" :key="message.id" class="message">
        <strong>{{ message.user.name }}:</strong> {{ message.content }}
      </div>
    </div>
    
    <div class="input-area">
      <input 
        v-model="newMessage" 
        @keyup.enter="sendMessage"
        placeholder="메시지를 입력하세요..."
        class="message-input"
      >
      <button @click="sendMessage" class="send-button">
        전송
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount, nextTick } from 'vue'
import axios from 'axios'
import Echo from 'laravel-echo'
import Pusher from 'pusher-js'

const messages = ref([])
const newMessage = ref('')
const messagesContainer = ref(null)
let echo = null

// Echo 초기화 함수
const initializeEcho = () => {
  echo = new Echo({
    broadcaster: 'reverb',
    key: import.meta.env.VITE_REVERB_KEY,
    wsHost: import.meta.env.VITE_REVERB_HOST,
    wsPort: import.meta.env.VITE_REVERB_PORT,
    forceTLS: false,
    disableStats: true,
    enabledTransports: ['ws', 'wss']
  })

  // 채널 구독
  echo.channel('chat')
    .listen('MessageSent', (e) => {
      messages.value.push(e.message)
      scrollToBottom()
    })
}

// 메시지 전송
const sendMessage = async () => {
  if (!newMessage.value.trim()) return

  try {
    const response = await axios.post('/api/messages', {
      content: newMessage.value
    })
    
    // 입력창 초기화
    newMessage.value = ''
    
    // 스크롤 이동
    await scrollToBottom()
  } catch (error) {
    console.error('메시지 전송 에러:', error)
  }
}

// 스크롤을 최하단으로 이동
const scrollToBottom = async () => {
  await nextTick()
  if (messagesContainer.value) {
    messagesContainer.value.scrollTop = messagesContainer.value.scrollHeight
  }
}

// 초기 메시지 로드
const loadInitialMessages = async () => {
  try {
    const response = await axios.get('/api/messages')
    messages.value = response.data
    await scrollToBottom()
  } catch (error) {
    console.error('초기 메시지 로드 에러:', error)
  }
}

onMounted(() => {
  initializeEcho()
  loadInitialMessages()
})

onBeforeUnmount(() => {
  if (echo) {
    echo.leaveChannel('chat')
  }
})
</script>

<style scoped>
.chat-container {
  @apply flex flex-col h-[500px] border rounded-lg;
}

.messages {
  @apply flex-grow overflow-y-auto p-4 space-y-2;
}

.message {
  @apply p-2 bg-gray-100 rounded;
}

.input-area {
  @apply flex gap-2 p-4 border-t;
}

.message-input {
  @apply flex-grow px-3 py-2 border rounded;
}

.send-button {
  @apply px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600;
}
</style>
```

## 2.4 Laravel 컨트롤러 및 이벤트

```php
// app/Events/MessageSent.php
namespace App\Events;

use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class MessageSent implements ShouldBroadcast
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    public $message;

    public function __construct($message)
    {
        $this->message = $message;
    }

    public function broadcastOn(): Channel
    {
        return new Channel('chat');
    }
}

// app/Http/Controllers/MessageController.php
namespace App\Http\Controllers;

use App\Events\MessageSent;
use App\Models\Message;
use Illuminate\Http\Request;

class MessageController extends Controller
{
    public function index()
    {
        return Message::with('user')
            ->latest()
            ->take(50)
            ->get()
            ->reverse()
            ->values();
    }

    public function store(Request $request)
    {
        $message = Message::create([
            'user_id' => auth()->id(),
            'content' => $request->content
        ]);

        $message->load('user');

        broadcast(new MessageSent($message))->toOthers();

        return response()->json($message);
    }
}
```

# 3. 성능 최적화 및 에러 처리

## 3.1 WebSocket 재연결 처리
```vue
<script setup>
// ReconnectingWebSocket.vue
const maxReconnectAttempts = 5
let reconnectAttempts = 0

const initializeEchoWithReconnect = () => {
  try {
    initializeEcho()
    reconnectAttempts = 0
  } catch (error) {
    if (reconnectAttempts < maxReconnectAttempts) {
      reconnectAttempts++
      const delay = Math.min(1000 * Math.pow(2, reconnectAttempts), 30000)
      setTimeout(initializeEchoWithReconnect, delay)
    } else {
      console.error('WebSocket 연결 실패')
    }
  }
}
</script>
```

## 3.2 메시지 최적화
```vue
<script setup>
import { computed } from 'vue'

// 가상 스크롤링을 위한 메시지 계산
const visibleMessages = computed(() => {
  const startIdx = Math.floor(scrollPosition.value / messageHeight)
  const count = Math.ceil(containerHeight / messageHeight)
  return messages.value.slice(startIdx, startIdx + count)
})
</script>
```

# 4. 보안 설정

## 4.1 채널 인증
```php
// routes/channels.php
Broadcast::channel('chat', function ($user) {
    return [
        'id' => $user->id,
        'name' => $user->name
    ];
});
```

## 4.2 CORS 설정
```php
// config/cors.php
return [
    'paths' => ['api/*', 'broadcasting/auth'],
    'allowed_methods' => ['*'],
    'allowed_origins' => [env('FRONTEND_URL', 'http://localhost:5173')],
    'allowed_origins_patterns' => [],
    'allowed_headers' => ['*'],
    'exposed_headers' => [],
    'max_age' => 0,
    'supports_credentials' => true,
];
```

이상이 Vue.js와 Laravel의 최신 버전을 사용한 실시간 통신 구현 가이드입니다. Vite와 최신 컴포지션 API를 활용하여 작성되었습니다.