<div align="center">

<img src="/images/miniatura.png" >

<img src='./images/icon.png' width='60px' />

# ⚽ ChatApp FULLSTACK | Real-time Chat APP with React, Nestjs GraphQL, WebSockets & Advanced Caching



> Fullstack application demonstrating advanced patterns,
  best practices, performance optimization enterprise-level architecture and cache-aside pattern

[![TypeScript](https://img.shields.io/badge/TypeScript-5.7-3178c6?logo=typescript)](https://www.typescriptlang.org/)
[![NestJS](https://img.shields.io/badge/NestJS-11-E0234E?logo=nestjs)](https://nestjs.com/)
[![GraphQL](https://img.shields.io/badge/GraphQL-Apollo-E10098?logo=graphql)](https://www.apollographql.com/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15-336791?logo=postgresql)](https://www.postgresql.org/)
[![Redis](https://img.shields.io/badge/Redis-7+-DC382D?logo=redis)](https://redis.io/)

</div>


## 🎯 What & Why
### What is Futbol Chat?
**Production-ready real-time chat API**. If you are passionate about futbol argentino and are looking to discuss teams, players and whatever you want, this is your place. Built to demonstrate enterprise-level software engineering skills. It´s not just a chat application--it´s a showcase of architectural thinking, performance optimization, security best practices, and scalable system design. 
### Why I Built This
This project was designed to demonstrate:
 - **Architectural Expertise** - Layered architecture, SOLID principles, and proven design patterns
 - **Performance Engineering** - Cache-Aside pattern with redis, achieving 75x performance improvement
 - **Security Consciousness** - JWT authentication, bcrypt hashing, and authorization guards
 - **Real-Time Systems** - WebSocket subscriptions and efficient Pub/Sub architecture with redis also
 - **Production Mindset** - Observability, error handling, rate limiting and Docker deployment
 - **Code Quality** - Type-safe development, comprehensive testing, and clean code principles

### The Challenge
Built a chat system that is:
- **Fast**
- **Secure**
- **Scalable**
- **Observable**
- **Maintainable**
   

## ⚡ Highlights
- 🟪 Layered Architecture
- 🟪 SOLID Principles + Design Patterns
- 🟨 Cache-Aside Pattern with Redis (50-100ms faster)
- 🟨 Cursor-Based Pagination
- ⬜ JWT Authentication + bcrypt hashing
- ⬜ HTTP-only Secure Cookies 
- ⬜ Distributed Rate Limiting (Redis-backed)
- 🟦 GraphQL + WebSockets (real-time features)
- 🟦 Live User Precense tracking with Redis Sets
- 🟦 Typing Indicators with efficient broadcasting
- 🟦 Pub/Sub Architecture for scalable event distribution
- 🟫 Structured Logging (Winston for Dev and Prod environments)
- 🟫 Correlation IDs for Request Tracing
- 🟫 Performance Metrics
- 🟫 Custom Error Formating

## 👁️ Pages Preview
<p align='center'>
<img src="/images/register-modal-portrait.png" width="40%"  style="max-width: 200px; min-width: 120px;">
<img src="/images/login-modal-portrait.png" width="40%"  style="max-width: 200px; min-width: 120px;">
<img src="/images/home-portrait.png" width="40%"  style="max-width: 200px; min-width: 120px;">
 </p>
 <p align='center'>
<img src="/images/crear-sala-modal-portrait.png" width="40%"  style="max-width: 200px; min-width: 120px;">
<img src="/images/mis-salas-modal-portrait.png" width="40%"  style="max-width: 200px; min-width: 120px;">

 </p>
  <p align='center'>
<img src="/images/user-config-modal-portrait.png" width="40%"  style="max-width: 200px; min-width: 120px;">
<img src="/images/editar-perfil-modal-portrait.png" width="40%"  style="max-width: 200px; min-width: 120px;">

 </p>
   <p align='center'>
<img src="/images/room-portrait.png" width="40%"  style="max-width: 200px; min-width: 120px;">
<img src="/images/editar-sala-modal-portrait.png" width="40%"  style="max-width: 200px; min-width: 120px;">

 </p>
  <p align='center'>
<img src="/images/agregar-usuarios-sala-modal-portrait.png" width="40%"  style="max-width: 200px; min-width: 120px;">
<img src="/images/eliminar-sala-modal-portrait.png" width="40%"  style="max-width: 200px; min-width: 120px;">

 </p>




 
## 🏗️ Architecture

### **Redis: The Real Backbone**
Redis didnt just store jobs -- it became our system´s communication hub:
 - Pub/Sub for real-time updates
 - Rate-limiting using Redis tokens
 - Caching frequently fetched data
Because Redis operates in memory, the performance gain was huge. For high-concurrency scenarios, it´s indispensable.

### Cache Aside Pattern
In modern-day software development speed is a non-negotiable performance matrix. And to enhance the performance of your software and increase the response speed caching is a must-have tool.

**Getting the tool tou need**
Before we roll up our sleeves and get into the code, lets ensure we have all the tools installes. You´ll need.
 - `ioredis`: This is the Redis client for Nodejs that lets us interact with Redis from our NestJS app.
 - Install with: `npm i ioredis`

**Lets Set Up Redis -- Building our Redis Service**
Now that we´ve got the dependencies in place, lets set up our Redis Service.
This service will be our one-stop shop for interacting with Redis. You'll be able to store, retrieve, and delete data with just a few lines of code.
 ```typescript
   import { Injectable, OnModuleDestroy } from '@nestjs/common';
import Redis from 'ioredis';
import { ErrorManager } from '../../utils/error.manager';

@Injectable()
export class RedisCacheService implements OnModuleDestroy {
  private readonly context = 'CacheService';
  private client: Redis;

  constructor() {
    this.client = new Redis({
      host: process.env.REDIS_HOST || 'localhost',
      port: parseInt(process.env.REDIS_PORT || '6379', 10),
      password: process.env.REDIS_PASSWORD,
    });
  }

  onModuleDestroy() {
    this.client.quit();
  }

  async set(key: string, value: unknown, ttlInSeconds: number): Promise<void> {
    const strValue = JSON.stringify(value);
    if (ttlInSeconds && ttlInSeconds > 0) {
      await this.client.set(key, strValue, 'EX', ttlInSeconds); // 'EX' sets an expiration time
    } else {
      await this.client.set(key, strValue);
    }
  }

  async get<T>(key: string): Promise<T | null> {
    const raw = await this.client.get(key);
    if (!raw) return null;
    try {
      // Usar reviver para convertir ISO strings de vuelta a Date objects
      return JSON.parse(raw, this.dateReviver) as T;
    } catch (error) {
      // const duration = Date.now() - startTime;
      //       this.logger.error(
      //         `Redis cache error, trying get data: ${error.message}`,
      //         error.stack,
      //         this.context,
      //         { duration },
      //       );
      throw ErrorManager.createSignatureError(error.message);
    }
  }

  /**
   * Reviver para JSON.parse que convierte ISO date strings a Date objects
   * Identifica strings que parecen ISO 8601 y los convierte a Date
   */
  private dateReviver = (key: string, value: unknown): unknown => {
    if (typeof value === 'string') {
      // Patrón ISO 8601: YYYY-MM-DDTHH:mm:ss.sssZ
      const isoDatePattern =
        /^\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}(\.\d{3})?Z?$/;
      if (isoDatePattern.test(value)) {
        return new Date(value);
      }
    }
    return value;
  };

  async del(key: string): Promise<number> {
    return await this.client.del(key);
  }
  // Delete keys by pattern using SCAN (safer than KEYS)
  async delByPattern(pattern: string): Promise<number> {
    let cursor = '0';
    let deleted = 0;
    do {
      const [nextCursor, keys] = await this.client.scan(
        cursor,
        'MATCH',
        pattern,
        'COUNT',
        '100',
      );
      cursor = nextCursor;
      if (keys.length) {
        const res = await this.client.del(...keys);
        deleted += res;
      }
    } while (cursor !== '0');
    return deleted;
  }

  // Helper wrap: try cache, otherwise call fn and cache result
  async wrap<T>(
    key: string,
    ttlSeconds: number,
    fn: () => Promise<T>,
  ): Promise<T> {
    const cached = await this.get<T>(key);
    if (cached !== null) return cached;
    const val = await fn();
    await this.set(key, val, ttlSeconds);
    return val;
  }
}
 ```
This service will help The Redis client connect to your Redis server when your app starts. If you’re running it locally or in Docker, this is super handy.
Also in this we’ve got a built-in cleanup when the app shuts down. Redis connections will close properly, so you don’t have any stray processes running.

Then we need weap this service in the module. Before i created a pubsub.module. for instance a RedisPubSub client and that it can be available around the application. So we gonna use it for export the RedisCacheService also.
 ```typescript
  import { RedisCacheService } from '@/src/common/cache/services/cache.service';
import { PUB_SUB } from '@/src/common/constants';
import { Global, Module } from '@nestjs/common';
import { RedisPubSub } from 'graphql-redis-subscriptions';
import Redis from 'ioredis';

/*

- Crear instancia RedisPubSub y exportarla como provider global
- Todos los resolvers/services compartiran la misma conexion a Redis
   asi evitar instancias duplicadas.

- Patrón Pub/Sub : productores PUBlican mensajes en un canal, y
  consumidores se SUBscriben a canales y reciben mensajes.

- Redis Pub/Sub: mecanismo muy rápido, basado en memoria, fire-and-forget.
   -No hay persistencia
   - No hay ack/offsets


*/

const host = process.env.REDIS_HOST || 'localhost';
const port = parseInt(process.env.REDIS_PORT || '6379', 10);
const password = process.env.REDIS_PASSWORD;

const pubSub = new RedisPubSub({
  connection: {
    host,
    port,
    password,
    retryStrategy: (times) => Math.min(times * 50, 2000),
  },
  publisher: new Redis({ host, port, password }),
  subscriber: new Redis({ host, port, password }),
});

@Global()
@Module({
  providers: [
    {
      provide: PUB_SUB,
      useValue: pubSub,
    },
    RedisCacheService, // <- here
  ],
  exports: [PUB_SUB, RedisCacheService], //<- here
})
export class PubSubModule {}
 ```
Now we can use the RedisCacheService and RedisPubSub, whereever we want in the app.
Remember import the PubSubModule in app.module.


Here is a simple example of how i use Redis for caching.
### Concept:
<img src="/images/cache-strategy.png" >

 ```typescript
   // Read-through caching strategy
const cacheKey = `cache:chatroom:messages:${chatroomId}`;

// Try cache first
const cached = await cacheService.get(cacheKey);
if (cached) return cached;

// Fallback to database
const data = await database.query();

// Populate cache for next request
await cacheService.set(cacheKey, data, 60); // 60s TTL
 ```
### Real-life use case
Check how i use it. In ChatroomResolver for getChatroomById Query.
 ```typescript
 export class ChatroomResolver {
  constructor(
    private readonly chatroomService: ChatroomService,
    private readonly cacheService: RedisCacheService,
  ) {}

  @Query(() => ChatroomEntity, {
    name: 'getChatroomById',
    description: 'Get a chatroom by id',
  })
  async getChatroomById(
    @Args('chatroomId') chatroomId: number,
    @Context() { req, correlationId }: GraphQLContext,
  ) {
    
    const cacheKey = `cache:chatroom:getById:${chatroomId}`;
    const cached = await this.cacheService.get<ChatroomEntity>(cacheKey);
    if (cached) {
      return cached;
    }
    const result = await this.chatroomService.getChatroom(chatroomId);
    if (result) {
      await this.cacheService.set(cacheKey, result, 60); //60 seg
    }
    return result;
  }
}
 ```
By leveraging Redis for caching, you can significantly reduce database load, speed up response times, and scale your application more effectively. Here’s a recap of the benefits:
- **Faster Response Times**: Cache database query results for frequently accessed data.
- **Reduced Database Load**: Avoid querying the database for the same data repeatedly.
- **Improved User Experience**: Deliver lightning-fast responses even under high traffic conditions.

Study guide: [How to Use Redis with NestJS: A Simple Guide to Caching](https://medium.com/@dipghoshraj/how-to-use-redis-with-nestjs-a-simple-guide-to-caching-b9408d96243e)


### Event-Driven Architecture
**Redis Pub/Sub**
Redis comes with a built-in Pub/Sub (publish/suscribe) system that lets applications talk to each other in real time. One app can publish a message and any other app thats suscribed to the same channel will instantly receive it.
<img src="/images/redis-pubsub-1.png" >
<img src="/images/redis-pubsub-2.png" >

Think of it like a radio station: the publisher is the radio tower, and suscribers are the listeners. If your radio is off, you miss the broadcast -- Redis doestn keep a history of messages.
**What is Redis Pub/Sub?**
At its core:
- Publisher sends messages to a chanel
- Subscribers listen and get the messages rigth away.
- No direct connection needed-publishers dont know who's listening, and subscribers dont care who´s sending.
- Real-time by design perfect for live updates and instant notifications.

Before in Cache section, i´ve show the pubsub.module with the instance. We'll use that.
So, let me show you how we can use this feature. We gonna create a mutation 'sendMessage', inside we publish the message.
Later we gonna create a Subscription to get the message.
 ```typescript

  @Mutation(() => MessageEdge)
  async sendMessage(
    @Args('chatroomId') chatroomId: number,
    @Args('content') content: string,
    @Context() { req, correlationId }: GraphQLContext,
  ): Promise<MessageEdge> {

    const newMessage = await this.chatroomService.sendMessage(
      chatroomId,
      content,
      req.user.sub,
    );
   
    if (!newMessage.node.createdAt) {
      newMessage.node.createdAt = new Date();
    }

    await this.pubSub
      .publish(`newMessage.${chatroomId}`, { newMessage }) //newMessage is the publish key
      .catch((err) => {
        this.logger.error(
          'Filed to publish event to Redis',
          err.stack,
          this.context,
          {
            event: NEW_MESSAGE,
            messageId: newMessage.cursor,
            error: err.message,
          },
        );
      });

    return newMessage;
  }

 ```
```typescript
//this graphql subscription must be called newMessage
@Subscription(() => MessageEdge, {
    nullable: true,
    resolve: (value) => {
      return value.newMessage;
    },
  })
  newMessage(@Args('chatroomId', { type: () => Int }) chatroomId: number) {
    this.logger.log(
      'New Subscription client connected: postCreated',
      this.context,
    );
    return this.pubSub.asyncIterableIterator(`newMessage.${chatroomId}`);//newMessage is the public key
  }
```
Just like that. The name of the graphql subscription must be the same of the publish key.
In this case the publish key is newMessage, so the method must be called newMessage.

```graphql
mutation SendMessage($chatroomId: Float!, $content: String!) {
    sendMessage(chatroomId: $chatroomId, content: $content) {
      node {
        id
        content
        createdAt
        user {
          id
          fullname
          avatarUrl
        }
      }
      cursor
    }
  }



subscription NewMessage($chatroomId: Int!) {
    newMessage(chatroomId: $chatroomId) {
      node {
        id
        content
        createdAt
        user {
          id
          fullname
          avatarUrl
        }
      }
      cursor
    }
  }


```


### Infrastructure Optimizations That Made a Difference
**Dockerized Everything**. Each NestJs instance, Main DB´s, and Redis server ran in its own container. This made autoscaling on Kubernetes a breeze.


## 🛠️ Technology Stack
### Core Framework

| Technology     | Version | Purpose                       | Key Packages |
| ---------------| --------|-------------------------------|--------------|
| NestJS         | 11x     | Progressive Node.js framework | `@nestjs`    | 
| TypeScript     | 5.7     | Type-safe development         | `typescript` |
| Node.js        | 22+     | JavaScript runtime            |              |

### API & Real-Time

| Technology     | Purpose                                | Key Packages |
| ---------------| ---------------------------------------|--------------|
| GraphqQL       | Type-safe API with Apollo Server       | `graphql`    | 
| Apollo Server  | Production-grade GraphQL server        | `@apollo/server` |
| WebSockets     | Bidirectional real-time communication  |             |
| grapqh-subscriptions     | Server-push subscriptions  |    `graphql-redis-subscriptions`          |

### Database & Storage

| Technology     | Purpose                                | Key Packages              |
| ---------------| ---------------------------------------|---------------------------|
| PostgreSQL 15  | ACID-compliant relational database     | Docker Image              | 
| Prisma ORM     | Type-safe database client & migrations | `prisma`,`@prisma/client` |
| Redis 7        | In-memory cache + Pub/Sub              |   Docker Image            |
| ioredis        | Server-push subscriptions              |    `ioredis`              |

### Security

| Technology      | Purpose                           | Key Packages      |
| ----------------| ----------------------------------|-------------------|
| JWT             | Stateless authentication tokens   | `@nestjs/jwt`     | 
| bcrypy          | Password hashing                  | `bcrypt`          |
| class-validator | Input validation decorators       | `class-validator` |
| Custom Guards   | File-grained authorization        |                   |

### Development & Quality


| Technology      | Purpose                           | Key Packages      |
| ----------------| ----------------------------------|-------------------|
| Jest            | Unit & integration testing        | `jest`            | 
| Supertest       | E2E API testing                   | `supertest`       |
| ESLint          | Code linting & standards          | `eslint`          |
| Prettier        | Code formatting                   |                   |
| Winston         | Structured logging                |  `winston`        |

### Frontend
| Technology                              | Purpose          | Key Packages                                        |
| ----------------------------------------|------------------|-----------------------------------------------------|
| React 19, TypeScript, Vite              | Frontend         | `react`, `react-router-dom`, `vite`                 | 
| React Compiler                          | Performance      | `babel-plugin-react-compiler`                       | 
| Tailwind CSS, Radix UI, Shadcn          | UI Framework     | `tailwindcss`, `@radix-ui/react-*`, `lucide-react,` |
| React Hook Form and Zod for Validations | Form             | `react-hook-form`, `zod`                            |
| Zustand                                 | State Management |           `zustand`                                 |


## ⭐ Features
### User Management
 - ✅ User registration with validation
 - ✅ Secure login with JWT tokens
 - ✅ Profile updates (including avatar upload)
 - ✅ User search functionality
 - ✅ Session management with refresh tokens

### Chatroom Operations
- ✅ Create public/private chatrooms
- ✅ Add/remove members with access control
- ✅ Chatroom metadata (name, description, color, image)
- ✅ List chatrooms per user with caching
- ✅ Delete chatrooms with cascade cleanup

### Messaging
- ✅ Send text messages with validation
- ✅ Image upload via GraphQL multipart
- ✅ Paginated message history (cursor-based)
- ✅ Real-time message delivery via subscriptions
- ✅ Message timestamps with timezone support

### Real-Time Features
- ✅ Live User Presence - Track active users per chatroom
- ✅ Typing Indicators - Show when users are typing
- ✅ Message Subscriptions - Instant message delivery
- ✅ User Events - Join/leave notifications
- ✅ Connection Management - WebSocket lifecycle hooks


## Performance & Optimization
### Caching Strategy :
[Cache Aside Pattern](#cache-aside-pattern)
### Database Optimization
**Cursor-Based Pagination**
Instead of loading large amounts of data on the frontend, the API returns a limited set of items along with a cursor that points to the next batch. This way the client only fetches what it needs, which reduces payload size and speeds up the UI.
```graphql
query GetMessagesForChatroom($chatroomId: Float!, $take: Int!, $cursor: Int) {
    getMessagesForChatroom(
      chatroomId: $chatroomId
      take: $take
      cursor: $cursor
    ) {
      edges {
        node {
          id
          content
          imageUrl
          createdAt
          user {
            id
            fullname
            avatarUrl
          }
        }
        cursor
      }
      pageInfo {
        hasNextPage
        endCursor
      }
      totalCount
    }
  }
```

### Rate Limiting
For this technical i use `@nestjs/throttler` that is recomended for nestjs oficial doc.
```typescript
ThrottlerModule.forRootAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (configService: ConfigService) => ({
        throttlers: [
          {
            name: 'long',
            ttl: seconds(60),
            limit: 100,
          },
        ],
        // Storage de Redis
        storage: new ThrottlerStorageRedisService({
          host: configService.get('REDIS_HOST', 'localhost'),
          port: configService.get('REDIS_PORT', 6379),
          password: configService.get('REDIS_PASSWORD'),

          // Opciones adicionales
          keyPrefix: 'throttler:', // Prefijo para keys

          // TLS para producción
          ...(configService.get('REDIS_TLS') === 'true' && {
            tls: {},
          }),
        }),
      }),
    }),


```
Custom Rate-Limiting for specific queries and mutations.
```typescript

//Mutations
@Mutation()
@Throttle({ short: { limit: 2, ttl: seconds(3) } })
async sendMessage() { ... }

@Mutation()
@Throttle({ short: { limit: 1, ttl: seconds(3) } }) // RateLimit mas estricto - 1 por 3 segundos para auth
async register() { ... }


//Queries
@Query()
@Throttle({ short: { limit: 10, ttl: seconds(5) } }) // 10 requests/5s
async getChatroomById(){ ... }

//Skip throttle para todas las subscripciones - se puede hacer por la GrapqhQLThrottlerGuard
   @Subscription()
  @SkipThrottle()
  newMessage() { ... }
```






## 📊 Monitoring & Observability

### Structured Logging

```json
{
  "service": "chatapp",
  "correlationId": "unique-id-per-request",
  "operationName": "sendMessage",
  "context": "ChatroomResolver",
  "level": "info",
  "message": "Message sent successfully",
  "userId": 123,
  "chatroomId": 456,
  "duration": 45,
  "cacheHit": false,
  "timestamp": "2025-11-10T20:00:00Z"
}
```

### Correlation ID Tracking

- Permite trazar un request a lo largo de todo el sistema
- Útil para debugging en entornos distribuidos
- Agregado en middleware


## 🔐 Seguridad Multinivel

### Autenticación

- JWT tokens con expiry
- Refresh token rotation
- Secure cookie storage

### Autorización

```typescript
@UseGuards(GraphqlAuthGuard)  // Usuario autenticado
@UseGuards(ChatroomAccessGuard)  // Miembro del chatroom
async sendMessage(...) { ... }
```

- ## 🚀 Quick Start

### Prerequisites

- Node.js 22+
- PostgreSQL 15+
- Redis 7+
- Docker & Docker Compose (optional)

### Installation

```bash
# 1. Clone repository
git clone https://github.com/ValentinZoia/chatapp-backend.git
cd chatapp-backend

# 2. Install dependencies
npm install

# 3. Setup environment variables
cp .env.example .env.development

# 4. Configure .env.development
# DATABASE_URL="postgresql://user:password@localhost:5432/chatapp"
# REDIS_URL="redis://localhost:6379"
# JWT_SECRET="your-secret-key"

# 5. Run database migrations
npm run prisma:migrate

# 6. (Optional) Seed database
npm run prisma:seed

# 7. Start development server - before start all docker services
npm run start:dev
```

### Docker Compose

```bash
# Start all services (PostgreSQL + Redis)
docker-compose up -d

# View logs
docker-compose logs -f

# Stop services
docker-compose down
```

## 📚 Project Structure

```
src/
├── modules/                # Feature modules
│   ├── auth/               # Authentication & JWT
│   ├── user/               # User management
│   ├── chatroom/           # Chatroom operations
│   ├── live-chatroom/      # Real-time tracking
│   └── PubSub/             # Redis Pub/Sub setup
├── common/                 # Shared utilities
│   ├── cache/              # Redis caching service
│   ├── constants/          # Constants for Inject dependencies
│   ├── filter/             # GraphQL excpetion filter
│   ├── guards/             # Custom Guards
│   ├── interceptors/       # Request interceptors
│   ├── middleware/         # Custom middleware
│   ├── plugins/            # Logger Plugin to catch graphql context
│   ├── throttler/          # GraphQLThrottlerGuard -> give to throttler the graphql context
│   ├── token/              # TokenService -> verify the refreshtoken
│   ├── types/              # GraphQLExecutionTypes
│   ├── interfaces/         # GraphQLContext, ILogger, etc
│   ├── logger/             # Logging configuration
│   └── utils/              # Helper functions
├── app.module.ts           # Root module
└── main.ts                 # Application entry

prisma/
├── schema.prisma           # Database schema
└── migrations/             # Migration files
```

## 🤝 Contributing

1. Fork repository
2. Create feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to branch (`git push origin feature/AmazingFeature`)
5. Open Pull Request

## 📝 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 📧 Contact

- **Author:** Valentín Zoia
- **Email:** valentinzoia@gmail.com
- **LinkedIn:** [Profile](https://linkedin.com/in/valentinzoia)
- **Portfolio:** [Website](https://valentinzoia.dev)

---

**Built with ❤️ and best practices**


