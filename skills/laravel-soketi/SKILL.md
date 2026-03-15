---
description: Integrate Laravel with Soketi WebSocket server for real-time broadcasting. Covers installation, Docker, Echo, channels, events, SSL, and production deployment.
---

# Laravel Soketi Integration Assistant

You are a Laravel real-time broadcasting specialist with Soketi. Help configure WebSocket communication using Soketi server, Laravel Broadcasting, and Laravel Echo.

## What is Soketi

Soketi is a high-performance, open-source WebSocket server built on uWebSockets.js. It implements the **Pusher Protocol v7**, making it a drop-in replacement for Pusher with zero vendor lock-in.

- 8.5x faster than Fastify, 10x faster than Socket.IO
- Runs on minimal resources (1 GB RAM, 1 CPU for 500+ connections)
- $5-$10/month on cloud vs Pusher's $49/month plan

## Quick Setup

### 1. Install Soketi

```bash
# Via npm (global)
npm install -g @soketi/soketi

# Start server (default: 127.0.0.1:6001)
soketi start
```

**Default credentials:** `app-id`, `app-key`, `app-secret`

### 2. Install Laravel Dependencies

```bash
# Backend
composer require pusher/pusher-php-server

# Frontend
pnpm add -D laravel-echo pusher-js

# Enable broadcasting
php artisan install:broadcasting
```

### 3. Configure .env

```env
BROADCAST_CONNECTION=pusher

PUSHER_APP_ID=app-id
PUSHER_APP_KEY=app-key
PUSHER_APP_SECRET=app-secret
PUSHER_HOST=127.0.0.1
PUSHER_PORT=6001
PUSHER_SCHEME=http
PUSHER_APP_CLUSTER=mt1

VITE_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
VITE_PUSHER_HOST="${PUSHER_HOST}"
VITE_PUSHER_PORT="${PUSHER_PORT}"
VITE_PUSHER_SCHEME="${PUSHER_SCHEME}"
VITE_PUSHER_APP_CLUSTER="${PUSHER_APP_CLUSTER}"
```

### 4. Configure config/broadcasting.php

```php
'connections' => [
    'pusher' => [
        'driver' => 'pusher',
        'key' => env('PUSHER_APP_KEY', 'app-key'),
        'secret' => env('PUSHER_APP_SECRET', 'app-secret'),
        'app_id' => env('PUSHER_APP_ID', 'app-id'),
        'options' => [
            'cluster' => env('PUSHER_APP_CLUSTER', 'mt1'),
            'host' => env('PUSHER_HOST', '127.0.0.1'),
            'port' => env('PUSHER_PORT', 6001),
            'scheme' => env('PUSHER_SCHEME', 'http'),
            'encrypted' => true,
            'useTLS' => env('PUSHER_SCHEME', 'http') === 'https',
        ],
    ],
],
```

### 5. Configure Laravel Echo (`resources/js/bootstrap.js`)

```javascript
import Echo from 'laravel-echo';
import Pusher from 'pusher-js';

window.Pusher = Pusher;

window.Echo = new Echo({
    broadcaster: 'pusher',
    key: import.meta.env.VITE_PUSHER_APP_KEY,
    cluster: import.meta.env.VITE_PUSHER_APP_CLUSTER ?? 'mt1',
    wsHost: import.meta.env.VITE_PUSHER_HOST ?? '127.0.0.1',
    wsPort: import.meta.env.VITE_PUSHER_PORT ?? 80,
    wssPort: import.meta.env.VITE_PUSHER_PORT ?? 443,
    forceTLS: (import.meta.env.VITE_PUSHER_SCHEME ?? 'https') === 'https',
    enabledTransports: ['ws', 'wss'],
    encrypted: true,
    disableStats: true,
});
```

**Importante:** `enabledTransports: ['ws', 'wss']` e obrigatorio. Soketi nao suporta XHR polling.

---

## Docker Setup

```yaml
# docker-compose.yml
soketi:
    image: 'quay.io/soketi/soketi:latest-16-alpine'
    environment:
        SOKETI_DEBUG: '1'
        SOKETI_METRICS_SERVER_PORT: '9601'
    ports:
        - '${SOKETI_PORT:-6001}:6001'
        - '${SOKETI_METRICS_SERVER_PORT:-9601}:9601'
    networks:
        - sail
```

**Docker .env:** Backend usa hostname do container, frontend usa localhost:

```env
PUSHER_HOST=soketi
VITE_PUSHER_HOST="127.0.0.1"
```

---

## Event Broadcasting

### Create Event

```bash
php artisan make:event OrderShipped
```

```php
<?php

namespace App\Events;

use App\Models\Order;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class OrderShipped implements ShouldBroadcast
{
    use Dispatchable, SerializesModels;

    public function __construct(
        public Order $order,
    ) {}

    public function broadcastOn(): array
    {
        return [
            new PrivateChannel('orders.'.$this->order->user_id),
        ];
    }

    // Opcional: customizar dados enviados
    public function broadcastWith(): array
    {
        return [
            'id' => $this->order->id,
            'status' => $this->order->status,
        ];
    }

    // Opcional: broadcast condicional
    public function broadcastWhen(): bool
    {
        return $this->order->value > 100;
    }
}
```

### Dispatch Event

```php
use App\Events\OrderShipped;

OrderShipped::dispatch($order);
```

---

## Channel Types

### Public Channel

```php
// Event
public function broadcastOn(): array
{
    return [new Channel('news')];
}

// Echo (JS)
Echo.channel('news').listen('NewsPosted', (e) => {
    console.log(e);
});
```

### Private Channel

```php
// Event
public function broadcastOn(): array
{
    return [new PrivateChannel('orders.'.$this->order->user_id)];
}

// Authorization (routes/channels.php)
Broadcast::channel('orders.{userId}', function ($user, $userId) {
    return (int) $user->id === (int) $userId;
});

// Echo (JS)
Echo.private(`orders.${userId}`).listen('OrderShipped', (e) => {
    console.log(e.order);
});
```

### Presence Channel

```php
// Event
public function broadcastOn(): array
{
    return [new PresenceChannel('chat.'.$this->roomId)];
}

// Authorization (routes/channels.php)
Broadcast::channel('chat.{roomId}', function ($user, $roomId) {
    if ($user->canJoinRoom($roomId)) {
        return ['id' => $user->id, 'name' => $user->name];
    }
});

// Echo (JS)
Echo.join(`chat.${roomId}`)
    .here((users) => { console.log('Online:', users); })
    .joining((user) => { console.log('Joined:', user); })
    .leaving((user) => { console.log('Left:', user); })
    .listen('NewMessage', (e) => { console.log(e); });
```

---

## Production: NGINX + SSL

### NGINX Config

```nginx
server {
    listen 443 ssl;
    server_name ws.myapp.com;

    ssl_certificate /etc/letsencrypt/live/ws.myapp.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/ws.myapp.com/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:6001;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
        proxy_read_timeout 86400;
    }
}
```

### Production .env

```env
PUSHER_HOST=ws.myapp.com
PUSHER_PORT=443
PUSHER_SCHEME=https

VITE_PUSHER_HOST=ws.myapp.com
VITE_PUSHER_PORT=443
VITE_PUSHER_SCHEME=https
```

---

## Process Management

### Supervisor

```ini
[program:soketi]
command=soketi start
directory=/opt/soketi
autostart=true
autorestart=true
stdout_logfile=/var/log/soketi-supervisor.log
```

### PM2

```bash
pm2 start soketi-pm2 -- start
```

## Troubleshooting

| Problema | Solucao |
|----------|---------|
| Evento nao broadcasted | Verificar se o event implementa `ShouldBroadcast` |
| Conexao falha no browser | Verificar `VITE_PUSHER_HOST` aponta para localhost (dev) ou dominio (prod) |
| 403 na autorizacao | Verificar `routes/channels.php` e se o user esta autenticado |
| XHR polling fallback | Adicionar `enabledTransports: ['ws', 'wss']` no Echo |
| SSL com self-signed cert | Downgrade `pusher/pusher-php-server` para 5.0.3 |

## Reference Files

- `~/.claude/skills/laravel/soketi-integration.md`
