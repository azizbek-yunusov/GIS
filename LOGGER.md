# Professional Cache Strategiyasi - Best Practices

Keling, professional loyihalarda **cache**, **WebSocket** va **auth** modullarini qanday integratsiya qilish kerakligini ko'rib chiqaylik.

## 1. Cache Strategiyasi - Qayerda Ishlatish Kerak?

### ✅ CACHE ISHLATILISHI SHART BO'LGAN JOYLAR:

#### **Auth Module:**
```typescript
// ✅ RECOMMENDED: Session/Token caching
class AuthService {
  async validateToken(token: string) {
    // 1. Avval cache'dan tekshir (super fast)
    const cached = await this.cacheService.get(`auth:token:${token}`);
    if (cached) return JSON.parse(cached);

    // 2. Agar cache'da yo'q bo'lsa, database'dan
    const user = await this.userRepository.findByToken(token);
    
    // 3. Cache'ga saqlash (keyingi requests tezroq bo'lishi uchun)
    await this.cacheService.set(
      `auth:token:${token}`,
      JSON.stringify(user),
      3600 // 1 hour
    );
    
    return user;
  }

  // ✅ Refresh token tracking
  async validateRefreshToken(token: string) {
    const key = `auth:refresh:${token}`;
    const isValid = await this.cacheService.get(key);
    return isValid === 'true';
  }

  // ✅ Logout - token invalidation
  async logout(token: string) {
    await this.cacheService.del(`auth:token:${token}`);
    // Blacklist qo'shish
    await this.cacheService.set(
      `auth:blacklist:${token}`,
      'true',
      86400 // 24 hours
    );
  }
}
```

#### **WebSocket Module:**
```typescript
// ✅ RECOMMENDED: Active connections tracking
class ConnectionManagerService {
  // User online/offline status
  async markUserOnline(userId: string, socketId: string) {
    await this.cacheService.set(
      `ws:user:${userId}:status`,
      'online',
      300 // 5 minutes (auto-expire)
    );
    
    // User's active sockets
    await this.cacheService.sadd(`ws:user:${userId}:sockets`, socketId);
  }

  // ✅ Room participants tracking
  async getRoomParticipants(roomId: string): Promise<string[]> {
    const cached = await this.cacheService.get(`ws:room:${roomId}:users`);
    if (cached) return JSON.parse(cached);

    const users = await this.roomRepository.getParticipants(roomId);
    await this.cacheService.set(
      `ws:room:${roomId}:users`,
      JSON.stringify(users),
      600 // 10 minutes
    );
    
    return users;
  }
}

// ✅ Message delivery tracking
class ChatGateway {
  async sendMessage(data: ChatMessageDto) {
    const message = await this.wsService.createMessage(data);
    
    // Pending delivery tracking
    const recipientIds = await this.getRoomParticipants(data.roomId);
    
    for (const recipientId of recipientIds) {
      await this.cacheService.sadd(
        `ws:user:${recipientId}:pending`,
        message.id
      );
    }
    
    this.server.to(`room:${data.roomId}`).emit('new_message', message);
  }
}
```

#### **Performance-Critical Queries:**
```typescript
// ✅ RECOMMENDED: Frequently accessed data
class UserService {
  async getUserProfile(userId: string) {
    const cacheKey = `user:profile:${userId}`;
    const cached = await this.cacheService.get(cacheKey);
    
    if (cached) return JSON.parse(cached);

    const profile = await this.userRepository.findOne({
      where: { id: userId },
      relations: ['role', 'department', 'branch']
    });

    await this.cacheService.set(
      cacheKey,
      JSON.stringify(profile),
      1800 // 30 minutes
    );

    return profile;
  }
}

// ✅ Permissions caching (JUDA MUHIM!)
class RoleService {
  async getUserPermissions(userId: string): Promise<string[]> {
    const cacheKey = `user:${userId}:permissions`;
    const cached = await this.cacheService.get(cacheKey);
    
    if (cached) return JSON.parse(cached);

    const permissions = await this.permissionRepository
      .createQueryBuilder('permission')
      .innerJoin('permission.roles', 'role')
      .innerJoin('role.users', 'user')
      .where('user.id = :userId', { userId })
      .select('permission.name')
      .getRawMany();

    await this.cacheService.set(
      cacheKey,
      JSON.stringify(permissions),
      3600 // 1 hour
    );

    return permissions;
  }
}
```

### ❌ CACHE ISHLATILMASLIGI KERAK BO'LGAN JOYLAR:

```typescript
// ❌ BAD: Financial transactions
class PaymentService {
  async processPayment(data: PaymentDto) {
    // ALWAYS from database - NO CACHE!
    const balance = await this.getBalance(data.userId);
    // ...
  }
}

// ❌ BAD: Audit logs
class AuditService {
  async logAction(action: string) {
    // ALWAYS write to database - NO CACHE!
    await this.auditRepository.save({...});
  }
}

// ❌ BAD: One-time sensitive data
class PasswordResetService {
  async verifyResetToken(token: string) {
    // Database only - cache bu yerda xavfli
    return await this.tokenRepository.findOne({...});
  }
}
```

## 2. Professional Cache Architecture

### **Multi-Layer Caching Strategy:**

```typescript
// src/modules/cache/cache.service.ts
import { Injectable, Logger } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import { Redis } from 'ioredis';
import { InjectRedis } from '@nestjs-modules/ioredis';

@Injectable()
export class CacheService {
  private readonly logger = new Logger(CacheService.name);
  private readonly localCache = new Map<string, { value: any; expiry: number }>();
  private readonly LOCAL_CACHE_TTL = 60; // 60 seconds

  constructor(
    @InjectRedis() private readonly redis: Redis,
    private readonly config: ConfigService,
  ) {}

  /**
   * L1: Local memory cache (super fast, per-instance)
   * L2: Redis cache (fast, shared across instances)
   */
  async get<T = any>(key: string): Promise<T | null> {
    try {
      // L1: Check local cache first
      const local = this.localCache.get(key);
      if (local && local.expiry > Date.now()) {
        this.logger.debug(`[L1 HIT] ${key}`);
        return local.value as T;
      }

      // L2: Check Redis
      const value = await this.redis.get(key);
      if (value) {
        this.logger.debug(`[L2 HIT] ${key}`);
        const parsed = JSON.parse(value);
        
        // Store in L1 for next time
        this.localCache.set(key, {
          value: parsed,
          expiry: Date.now() + (this.LOCAL_CACHE_TTL * 1000),
        });
        
        return parsed as T;
      }

      this.logger.debug(`[CACHE MISS] ${key}`);
      return null;
    } catch (error) {
      this.logger.error(`Cache GET error: ${error.message}`);
      return null;
    }
  }

  async set(key: string, value: any, ttl: number = 3600): Promise<void> {
    try {
      const serialized = JSON.stringify(value);
      
      // Set in Redis
      await this.redis.setex(key, ttl, serialized);
      
      // Set in local cache too
      this.localCache.set(key, {
        value,
        expiry: Date.now() + Math.min(ttl, this.LOCAL_CACHE_TTL) * 1000,
      });
      
      this.logger.debug(`[CACHE SET] ${key} (TTL: ${ttl}s)`);
    } catch (error) {
      this.logger.error(`Cache SET error: ${error.message}`);
    }
  }

  async del(key: string | string[]): Promise<void> {
    try {
      const keys = Array.isArray(key) ? key : [key];
      
      // Delete from Redis
      await this.redis.del(...keys);
      
      // Delete from local cache
      keys.forEach(k => this.localCache.delete(k));
      
      this.logger.debug(`[CACHE DEL] ${keys.join(', ')}`);
    } catch (error) {
      this.logger.error(`Cache DEL error: ${error.message}`);
    }
  }

  // Pattern-based deletion
  async delPattern(pattern: string): Promise<void> {
    try {
      const keys = await this.redis.keys(pattern);
      if (keys.length > 0) {
        await this.del(keys);
      }
    } catch (error) {
      this.logger.error(`Cache DEL pattern error: ${error.message}`);
    }
  }

  // Set operations (for WebSocket tracking)
  async sadd(key: string, ...members: string[]): Promise<void> {
    await this.redis.sadd(key, ...members);
  }

  async srem(key: string, ...members: string[]): Promise<void> {
    await this.redis.srem(key, ...members);
  }

  async smembers(key: string): Promise<string[]> {
    return await this.redis.smembers(key);
  }

  async sismember(key: string, member: string): Promise<boolean> {
    return (await this.redis.sismember(key, member)) === 1;
  }
}
```

### **Cache Decorator (Reusable):**

```typescript
// src/modules/cache/decorators/cacheable.decorator.ts
import { SetMetadata } from '@nestjs/common';

export const CACHEABLE_KEY = 'cacheable';

export interface CacheableOptions {
  ttl?: number;
  keyPrefix?: string;
  keyGenerator?: (...args: any[]) => string;
}

export const Cacheable = (options: CacheableOptions = {}) => {
  return SetMetadata(CACHEABLE_KEY, options);
};

// Usage:
class UserService {
  @Cacheable({ 
    ttl: 1800,
    keyPrefix: 'user:profile',
    keyGenerator: (userId) => userId
  })
  async getUserProfile(userId: string) {
    return await this.userRepository.findOne({ where: { id: userId } });
  }
}
```

### **Cache Interceptor:**

```typescript
// src/modules/cache/interceptors/cache.interceptor.ts
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
} from '@nestjs/common';
import { Observable, of } from 'rxjs';
import { tap } from 'rxjs/operators';
import { Reflector } from '@nestjs/core';
import { CacheService } from '../cache.service';
import { CACHEABLE_KEY, CacheableOptions } from '../decorators/cacheable.decorator';

@Injectable()
export class CacheInterceptor implements NestInterceptor {
  constructor(
    private readonly cacheService: CacheService,
    private readonly reflector: Reflector,
  ) {}

  async intercept(
    context: ExecutionContext,
    next: CallHandler,
  ): Promise<Observable<any>> {
    const options = this.reflector.get<CacheableOptions>(
      CACHEABLE_KEY,
      context.getHandler(),
    );

    if (!options) {
      return next.handle();
    }

    const args = context.getArgs();
    const cacheKey = this.generateKey(options, args);

    // Try to get from cache
    const cachedValue = await this.cacheService.get(cacheKey);
    if (cachedValue !== null) {
      return of(cachedValue);
    }

    // If not in cache, execute and cache the result
    return next.handle().pipe(
      tap(async (response) => {
        await this.cacheService.set(
          cacheKey,
          response,
          options.ttl || 3600,
        );
      }),
    );
  }

  private generateKey(options: CacheableOptions, args: any[]): string {
    const prefix = options.keyPrefix || 'cache';
    
    if (options.keyGenerator) {
      return `${prefix}:${options.keyGenerator(...args)}`;
    }

    // Default: use first argument as key
    return `${prefix}:${args[0]}`;
  }
}
```

## 3. Auth + Cache Integration

```typescript
// src/modules/auth/auth.service.ts
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import { CacheService } from '../cache/cache.service';

@Injectable()
export class AuthService {
  constructor(
    private readonly jwtService: JwtService,
    private readonly cacheService: CacheService,
  ) {}

  async login(user: User) {
    const payload = { userId: user.id, role: user.role };
    const token = this.jwtService.sign(payload);

    // Cache user session
    await this.cacheService.set(
      `auth:session:${user.id}`,
      { token, ...payload },
      86400 // 24 hours
    );

    return { token };
  }

  async validateToken(token: string) {
    // 1. Check blacklist first (super important!)
    const isBlacklisted = await this.cacheService.get(`auth:blacklist:${token}`);
    if (isBlacklisted) {
      throw new UnauthorizedException('Token has been revoked');
    }

    // 2. Verify JWT
    try {
      const payload = this.jwtService.verify(token);
      
      // 3. Check cache for user data
      const cacheKey = `auth:user:${payload.userId}`;
      let user = await this.cacheService.get(cacheKey);

      if (!user) {
        // 4. If not cached, get from DB
        user = await this.userRepository.findOne({
          where: { id: payload.userId },
          relations: ['role', 'permissions'],
        });

        // 5. Cache for next time
        await this.cacheService.set(cacheKey, user, 3600);
      }

      return user;
    } catch (error) {
      throw new UnauthorizedException('Invalid token');
    }
  }

  async logout(userId: string, token: string) {
    // 1. Remove session
    await this.cacheService.del(`auth:session:${userId}`);
    
    // 2. Blacklist token
    await this.cacheService.set(
      `auth:blacklist:${token}`,
      'true',
      86400 // 24 hours
    );

    // 3. Clear user cache
    await this.cacheService.delPattern(`auth:user:${userId}*`);
    
    // 4. Clear permissions cache
    await this.cacheService.del(`user:${userId}:permissions`);
  }

  // Logout from all devices
  async logoutAll(userId: string) {
    await this.cacheService.delPattern(`auth:session:${userId}*`);
    await this.cacheService.delPattern(`auth:user:${userId}*`);
    await this.cacheService.del(`user:${userId}:permissions`);
  }
}
```

## 4. WebSocket + Cache Integration

```typescript
// src/modules/websocket/gateways/chat.gateway.ts
@WebSocketGateway({ namespace: '/chat' })
export class ChatGateway extends BaseGateway {
  constructor(
    private readonly cacheService: CacheService,
    // ... other dependencies
  ) {
    super();
  }

  async handleConnection(client: Socket) {
    const userId = client.data.userId;

    // 1. Mark user online in cache
    await this.cacheService.set(
      `ws:user:${userId}:status`,
      'online',
      300 // Auto-expire in 5 minutes
    );

    // 2. Add socket to user's active connections
    await this.cacheService.sadd(
      `ws:user:${userId}:sockets`,
      client.id
    );

    // 3. Get user's rooms from cache
    const rooms = await this.getUserRooms(userId);
    
    // 4. Join all rooms
    for (const room of rooms) {
      client.join(`room:${room.id}`);
    }

    // 5. Broadcast online status
    this.server.emit('user:status', {
      userId,
      status: 'online',
      timestamp: new Date(),
    });
  }

  async handleDisconnect(client: Socket) {
    const userId = client.data.userId;

    // 1. Remove this socket
    await this.cacheService.srem(
      `ws:user:${userId}:sockets`,
      client.id
    );

    // 2. Check if user still has other active connections
    const activeSockets = await this.cacheService.smembers(
      `ws:user:${userId}:sockets`
    );

    // 3. If no more active connections, mark offline
    if (activeSockets.length === 0) {
      await this.cacheService.set(
        `ws:user:${userId}:status`,
        'offline',
        86400 // Keep for 24 hours
      );

      this.server.emit('user:status', {
        userId,
        status: 'offline',
        timestamp: new Date(),
      });
    }
  }

  @SubscribeMessage('chat:send_message')
  async handleMessage(
    @MessageBody() dto: ChatMessageDto,
    @WsUser() userId: string,
  ) {
    // 1. Save message to DB
    const message = await this.wsService.createMessage(dto);

    // 2. Get room participants from cache
    const participants = await this.getRoomParticipants(dto.roomId);

    // 3. Track message delivery status in cache
    for (const participantId of participants) {
      if (participantId !== userId) {
        // Add to user's pending messages
        await this.cacheService.sadd(
          `ws:user:${participantId}:pending`,
          message.id
        );
      }
    }

    // 4. Send to room
    this.server.to(`room:${dto.roomId}`).emit('chat:new_message', message);

    // 5. Invalidate cache
    await this.cacheService.del(`room:${dto.roomId}:recent_messages`);
  }

  // Cache-optimized method
  private async getRoomParticipants(roomId: string): Promise<string[]> {
    const cacheKey = `room:${roomId}:participants`;
    const cached = await this.cacheService.get<string[]>(cacheKey);

    if (cached) return cached;

    const participants = await this.roomRepository
      .createQueryBuilder('room')
      .leftJoinAndSelect('room.participants', 'user')
      .where('room.id = :roomId', { roomId })
      .select('user.id')
      .getRawMany();

    const participantIds = participants.map(p => p.user_id);

    await this.cacheService.set(cacheKey, participantIds, 600);

    return participantIds;
  }
}
```

## 5. Cache Invalidation Strategy

```typescript
// src/modules/cache/cache-invalidation.service.ts
import { Injectable } from '@nestjs/common';
import { CacheService } from './cache.service';

@Injectable()
export class CacheInvalidationService {
  constructor(private readonly cacheService: CacheService) {}

  // User updated
  async invalidateUser(userId: string) {
    await this.cacheService.delPattern(`user:${userId}:*`);
    await this.cacheService.delPattern(`auth:user:${userId}*`);
  }

  // Role/Permission updated
  async invalidatePermissions(userId?: string) {
    if (userId) {
      await this.cacheService.del(`user:${userId}:permissions`);
    } else {
      // Invalidate all users' permissions
      await this.cacheService.delPattern(`user:*:permissions`);
    }
  }

  // Room updated
  async invalidateRoom(roomId: string) {
    await this.cacheService.delPattern(`room:${roomId}:*`);
  }

  // Message sent
  async invalidateRoomMessages(roomId: string) {
    await this.cacheService.del(`room:${roomId}:recent_messages`);
  }
}

// Usage in services:
class UserService {
  async updateUser(userId: string, data: UpdateUserDto) {
    const updated = await this.userRepository.update(userId, data);
    
    // Invalidate cache
    await this.cacheInvalidation.invalidateUser(userId);
    
    return updated;
  }
}
```

## 6. Cache Configuration

```typescript
// src/modules/cache/cache.module.ts
import { Module, Global } from '@nestjs/common';
import { RedisModule } from '@nestjs-modules/ioredis';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { CacheService } from './cache.service';
import { CacheInvalidationService } from './cache-invalidation.service';

@Global()
@Module({
  imports: [
    RedisModule.forRootAsync({
      imports: [ConfigModule],
      useFactory: (config: ConfigService) => ({
        type: 'single',
        url: config.get('REDIS_URL'),
        options: {
          password: config.get('REDIS_PASSWORD'),
          db: config.get('REDIS_DB', 0),
          keyPrefix: config.get('REDIS_KEY_PREFIX', 'app:'),
          retryStrategy: (times) => {
            return Math.min(times * 50, 2000);
          },
          maxRetriesPerRequest: 3,
        },
      }),
      inject: [ConfigService],
    }),
  ],
  providers: [CacheService, CacheInvalidationService],
  exports: [CacheService, CacheInvalidationService],
})
export class CacheModule {}
```

## Xulosa - Qachon Cache Ishlatish Kerak?

### ✅ CACHE SHART:
1. **Auth**: Token validation, permissions, sessions
2. **WebSocket**: User status, active connections, room participants
3. **Frequently read data**: User profiles, roles, settings
4. **Expensive queries**: Complex joins, aggregations
5. **Rate limiting**: Request tracking

### ❌ CACHE ISHLATMASLIK:
1. **Financial data**: Payments, balances
2. **Audit logs**: Security-critical data
3. **One-time tokens**: Password reset, verification
4. **Frequently changing data**: Real-time analytics
5. **Large objects**: Files, images (use CDN instead)

### 🎯 Professional Tips:
- **TTL**: Har doim expire time qo'ying
- **Invalidation**: Data o'zgarganda cache'ni tozalang
- **Monitoring**: Cache hit/miss rate kuzating
- **Fallback**: Cache fail bo'lsa, DB'ga o'ting
- **Namespacing**: Key'larga prefix qo'ying (`user:`, `ws:`, `auth:`)

Qo'shimcha savollar bo'lsa, so'rang!
