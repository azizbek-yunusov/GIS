# NestJS Terminus - Professional Health Check Yechimi

`@nestjs/terminus` - bu NestJS ilovalaringiz uchun professional health check (tizim salomatligi tekshiruvi) yechimi. Keling, keng qamrovli tushuntirishni boshlaymiz.

## Terminus nima va nega kerak?

Terminus mikroservislar va katta ilovalar uchun health check endpointlarini yaratish uchun mo'ljallangan. Bu sizga quyidagilarni tekshirish imkonini beradi:

- **Database ulanishi** - PostgreSQL, MongoDB, TypeORM
- **Disk xotirasi** - qancha joy qolgan
- **Memory (RAM)** - xotira sarfi
- **Mikroservislar** - boshqa servislar bilan bog'lanish
- **Custom checks** - o'z tekshiruvlaringiz

## O'rnatish

```bash
npm install @nestjs/terminus
```

Database tekshiruvlari uchun qo'shimcha paketlar:

```bash
# TypeORM uchun
npm install @nestjs/typeorm typeorm

# Mongoose uchun  
npm install @nestjs/mongoose mongoose

# Sequelize uchun
npm install @nestjs/sequelize sequelize
```

## Basic Health Check - Oddiy Misol

```typescript
// health.controller.ts
import { Controller, Get } from '@nestjs/common';
import { HealthCheck, HealthCheckService, HttpHealthIndicator } from '@nestjs/terminus';

@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private http: HttpHealthIndicator,
  ) {}

  @Get()
  @HealthCheck()
  check() {
    return this.health.check([
      // Oddiy HTTP tekshiruvi
      () => this.http.pingCheck('nestjs-docs', 'https://docs.nestjs.com'),
    ]);
  }
}
```

**Natija:**
```json
{
  "status": "ok",
  "info": {
    "nestjs-docs": {
      "status": "up"
    }
  },
  "error": {},
  "details": {
    "nestjs-docs": {
      "status": "up"
    }
  }
}
```

## Database Health Check - Ma'lumotlar Bazasi

### TypeORM bilan

```typescript
// health.controller.ts
import { Controller, Get } from '@nestjs/common';
import { 
  HealthCheck, 
  HealthCheckService, 
  TypeOrmHealthIndicator,
  MemoryHealthIndicator,
  DiskHealthIndicator
} from '@nestjs/terminus';

@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private db: TypeOrmHealthIndicator,
    private memory: MemoryHealthIndicator,
    private disk: DiskHealthIndicator,
  ) {}

  @Get()
  @HealthCheck()
  check() {
    return this.health.check([
      // Database tekshiruvi
      () => this.db.pingCheck('database'),
      
      // Memory tekshiruvi - 150MB dan oshmasligi kerak
      () => this.memory.checkHeap('memory_heap', 150 * 1024 * 1024),
      
      // Disk tekshiruvi - 50% dan ko'p bo'sh joy bo'lishi kerak
      () => this.disk.checkStorage('storage', { 
        path: '/', 
        thresholdPercent: 0.5 
      }),
    ]);
  }
}
```

### MongoDB (Mongoose) bilan

```typescript
import { MongooseHealthIndicator } from '@nestjs/terminus';

@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private mongo: MongooseHealthIndicator,
  ) {}

  @Get()
  @HealthCheck()
  check() {
    return this.health.check([
      () => this.mongo.pingCheck('mongodb'),
    ]);
  }
}
```

## Custom Health Indicator - O'zingizniki

Maxsus tekshiruvlar yaratish:

```typescript
// dog.health.ts
import { Injectable } from '@nestjs/common';
import { HealthIndicator, HealthIndicatorResult, HealthCheckError } from '@nestjs/terminus';

@Injectable()
export class DogHealthIndicator extends HealthIndicator {
  async isHealthy(key: string): Promise<HealthIndicatorResult> {
    const dogs = await this.checkDogs(); // Sizning logikangiz
    
    const isHealthy = dogs.length > 0;
    const result = this.getStatus(key, isHealthy, { count: dogs.length });

    if (isHealthy) {
      return result;
    }
    
    throw new HealthCheckError('Dogcheck failed', result);
  }

  private async checkDogs() {
    // Bu yerda itlarni tekshirish logikasi
    return ['buddy', 'charlie', 'max'];
  }
}
```

**Ishlatish:**

```typescript
@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private dogHealth: DogHealthIndicator,
  ) {}

  @Get()
  @HealthCheck()
  check() {
    return this.health.check([
      () => this.dogHealth.isHealthy('dogs'),
    ]);
  }
}
```

## Professional Setup - To'liq Konfiguratsiya

```typescript
// health.module.ts
import { Module } from '@nestjs/common';
import { TerminusModule } from '@nestjs/terminus';
import { HttpModule } from '@nestjs/axios';
import { HealthController } from './health.controller';
import { DogHealthIndicator } from './dog.health';

@Module({
  imports: [
    TerminusModule,
    HttpModule, // HTTP tekshiruvlar uchun
  ],
  controllers: [HealthController],
  providers: [DogHealthIndicator],
})
export class HealthModule {}
```

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { HealthModule } from './health/health.module';

@Module({
  imports: [
    HealthModule,
    // boshqa modullar...
  ],
})
export class AppModule {}
```

## Real World Example - Amaliy Misol

Katta loyihalar uchun kompleks health check:

```typescript
@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private db: TypeOrmHealthIndicator,
    private mongo: MongooseHealthIndicator,
    private memory: MemoryHealthIndicator,
    private disk: DiskHealthIndicator,
    private http: HttpHealthIndicator,
  ) {}

  // Barcha tekshiruvlar
  @Get()
  @HealthCheck()
  check() {
    return this.health.check([
      () => this.db.pingCheck('postgres'),
      () => this.mongo.pingCheck('mongodb'),
      () => this.memory.checkHeap('memory_heap', 300 * 1024 * 1024),
      () => this.memory.checkRSS('memory_rss', 300 * 1024 * 1024),
      () => this.disk.checkStorage('disk', { path: '/', thresholdPercent: 0.7 }),
    ]);
  }

  // Faqat database tekshiruvi
  @Get('db')
  @HealthCheck()
  checkDatabase() {
    return this.health.check([
      () => this.db.pingCheck('postgres'),
      () => this.mongo.pingCheck('mongodb'),
    ]);
  }

  // Faqat tashqi servislar
  @Get('external')
  @HealthCheck()
  checkExternal() {
    return this.health.check([
      () => this.http.pingCheck('payment-api', 'https://api.payment.com/health'),
      () => this.http.pingCheck('notification-api', 'https://api.notify.com/health'),
    ]);
  }

  // Liveness probe - Kubernetes uchun
  @Get('live')
  @HealthCheck()
  checkLive() {
    return this.health.check([]);
  }

  // Readiness probe - Kubernetes uchun
  @Get('ready')
  @HealthCheck()
  checkReady() {
    return this.health.check([
      () => this.db.pingCheck('postgres'),
    ]);
  }
}
```

## Kubernetes Integration

```yaml
# deployment.yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: myapp
    image: myapp:1.0
    livenessProbe:
      httpGet:
        path: /health/live
        port: 3000
      initialDelaySeconds: 30
      periodSeconds: 10
    readinessProbe:
      httpGet:
        path: /health/ready
        port: 3000
      initialDelaySeconds: 5
      periodSeconds: 5
```

## Redis Health Check

```typescript
import { Injectable } from '@nestjs/common';
import { HealthIndicator, HealthIndicatorResult } from '@nestjs/terminus';
import { RedisService } from './redis.service';

@Injectable()
export class RedisHealthIndicator extends HealthIndicator {
  constructor(private redisService: RedisService) {
    super();
  }

  async isHealthy(key: string): Promise<HealthIndicatorResult> {
    try {
      await this.redisService.ping();
      return this.getStatus(key, true);
    } catch (error) {
      return this.getStatus(key, false, { message: error.message });
    }
  }
}
```

## Best Practices - Eng Yaxshi Amaliyotlar

### 1. Timeout qo'ying
```typescript
() => this.http.pingCheck('api', 'https://api.com', { 
  timeout: 5000 // 5 soniya
})
```

### 2. Graceful degradation
```typescript
@Get()
@HealthCheck()
async check() {
  try {
    return await this.health.check([
      () => this.db.pingCheck('database'),
    ]);
  } catch (error) {
    // Xatolikni log qilish
    return { status: 'error', message: error.message };
  }
}
```

### 3. Monitoring bilan integratsiya
```typescript
import { Logger } from '@nestjs/common';

@Controller('health')
export class HealthController {
  private readonly logger = new Logger(HealthController.name);

  @Get()
  @HealthCheck()
  async check() {
    const result = await this.health.check([...]);
    
    if (result.status === 'error') {
      this.logger.error('Health check failed', result);
      // Prometheus, Grafana yoki boshqa monitoring toolga yuborish
    }
    
    return result;
  }
}
```

## Tavsiyalar

1. **Turli endpointlar yarating** - `/health`, `/health/db`, `/health/ready`
2. **Timeout belgilang** - tashqi servislar uchun
3. **Monitoring qo'shing** - Prometheus, Grafana
4. **Kubernetes probes ishlatng** - liveness va readiness
5. **Custom indicators yarating** - biznes logika uchun
6. **Logging qo'shing** - xatoliklarni kuzatish uchun

Bu yechim production-ready va katta loyihalar uchun mos keladi!
