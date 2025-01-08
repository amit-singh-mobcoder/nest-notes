# nest-notes

# How to implement Global Exception Handler

### Step 1: create global-exception.filter.ts
```javascript
// src/common/filters/global-exception.filter.ts
import { ArgumentsHost, ExceptionFilter, Global } from '@nestjs/common';

@Global()
export class GlobalExceptionFilter implements ExceptionFilter {
  catch(exception: any, host: ArgumentsHost) {
    const context = host.switchToHttp();
    const request = context.getRequest();
    const response = context.getResponse();

    const NODE_ENV: string = 'production';
    const { status, message } = exception;

    response.status(status).json({
      success: false,
      error: {
        statusCode: status,
        message: message || 'Internal Server Error',
        path: request.url,
        stack: NODE_ENV == 'development' ? exception.stack : 'undefined',
      },
    });
  }
}
```

### Step 2: Import the global exception filter in main.ts
```javascript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { GlobalExceptionFilter } from './common/helpers/global-exception.filter';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalFilters(new GlobalExceptionFilter())
  await app.listen(process.env.PORT ?? 3000);
}
bootstrap();

```
### Step 3: Hint
```javascript
// if you want to use on router handler directly, then it is also possible
@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get()
  @UseFilters(GlobalExceptionFilter) 
  getHello(): string {
    return this.appService.getHello();
  }
}

// if you are using like this, then we have no need to change main.ts file

/**
 Prefer applying filters by using classes instead of instances when possible. It reduces memory usage since Nest can easily reuse instances of the same class across your entire module.
*/
```

# NESTJS MICROSERVICE

# PATTERN IN NEST JS
In **NestJS Microservices**, a **pattern** refers to a key or identifier that is used to define the type of message being sent or received in a message-based communication. Patterns are critical in message-driven architectures because they determine how messages are routed to the appropriate handlers.

## Key Concepts of Patterns in NestJS Microservices
### 1. Message Routing
- When a microservice receives a message, it uses the pattern to determine which handler should process the message.
- Patterns act as a mapping between messages and their respective handlers.

### 2. Patterns as Keys
- Patterns are typically strings, numbers, or even JSON objects that uniquely identify the type of message or event.
```javascript
@MessagePattern('get_user')
getUser(data: any) {
  return { id: data.id, name: 'John Doe' };
}
```
### 3. Patterns in Request-Response Communication
- In a request-response setup, the pattern defines the type of request being sent.
```javascript
const response = await client.send('get_user', { id: 1 }).toPromise();
```
### 4. Patterns in Event-Based Communication
- For event-driven systems, patterns define the type of event being published or subscribed to.
```javascript
@EventPattern('user_created')
handleUserCreatedEvent(data: any) {
  console.log('User created:', data);
}
```

## Common Use Cases of Patterns
**1. Request-Response**
- The client sends a message with a specific pattern, and the server responds based on that pattern.
```javascript
// Client
client.send('get_user', { id: 1 });

// Server
@MessagePattern('get_user')
getUser(data: any) {
  return { id: data.id, name: 'Jane Doe' };
}
```

**2. Event Broadcasting**
- Patterns are used to subscribe to specific events.
```javascript
// Publisher
client.emit('order_created', { orderId: 123 });

// Subscriber
@EventPattern('order_created')
handleOrderCreated(data: any) {
  console.log('Order created:', data);
}
```

**3. Custom Patterns**
- Patterns can be more complex, such as JSON objects, allowing for more specific routing.
```javascript
@MessagePattern({ cmd: 'get_user', role: 'admin' })
getAdminUser(data: any) {
  return { id: data.id, name: 'Admin User' };
}
```

## How Patterns Work Internally
**Client-Side**
- The client sends a message with a specific pattern to the message broker (e.g., Kafka, RabbitMQ, Redis).

**Message Broker**
- The broker routes the message to the appropriate microservice(s) based on the pattern.

**Server-Side**
- The microservice listens for messages with specific patterns and executes the corresponding handler.

Patterns in NestJS microservices are essential for defining message routing in both request-response and event-driven communication. They allow developers to create flexible and scalable systems by mapping messages to the appropriate handlers based on predefined keys or identifiers.

