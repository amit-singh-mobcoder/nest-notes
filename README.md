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
