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
