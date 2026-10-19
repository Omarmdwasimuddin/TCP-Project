## NestJS Microservices with TCP Transport

A hands-on project demonstrating TCP-based communication between `auth-service` and `order-service` using NestJS Microservices.

### Topics Covered
- TCP Transport
- `ClientProxy` and `ClientsModule`
- `@MessagePattern()` and `@Payload()`
- Request-response communication using `send()`
- Inter-service user validation

[Nx Monorepo Setup](https://github.com/Omarmdwasimuddin/Nx-Monorepo-Setup)

#### Install Microservices package
```bash
npm i @nestjs/microservices@^11
```
---

#### `auth-service/src/main.ts`
```ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app/app.module';
import { Transport, MicroserviceOptions } from '@nestjs/microservices';

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(AppModule, {
    transport: Transport.TCP,
    options: {
      host: '127.0.0.1',
      port: 5005,

    }
  })
    await app.listen();

    console.log('Auth microservice is listening on port 5005');
}

bootstrap();
```
---

#### `auth-service/src/app/app.controller.ts`
```ts
import { Controller } from '@nestjs/common';
import { AppService } from './app.service';
import { MessagePattern, Payload } from '@nestjs/microservices';


@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @MessagePattern({ cmd: 'validate_user' })
  handleUserValidation(@Payload() data: any){

    console.log('Auth service received data: ', data);

    if(data.userId === 1){
      return { status: 'success', user: {
        id: 1, 
        name: 'Wasim',
      }}
    }
    return { status: 'error', message: 'User not found!' };
  }
}
```
---


#### `order-service/src/app/app.module.ts`
```ts
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';

import { Transport, ClientsModule } from '@nestjs/microservices';

@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'AUTH_SERVICE',
      transport: Transport.TCP,
      options: {
        host: '127.0.0.1',
        port: 5005,
      }
      }
    ])
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```
---


#### `order-service/src/app/app.controller.ts`
```ts
import { Controller, Get, Inject, Param } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';
import { firstValueFrom } from 'rxjs';

@Controller('orders')
export class AppController {
  constructor(@Inject('AUTH_SERVICE') private readonly authClient: ClientProxy) {}

  @Get(':id')
  async createOrder(@Param('id') userId: string){
    const pattern = { cmd: 'validate_user' };
    const payload = { userId: Number(userId) };
    const authResponse = await firstValueFrom(this.authClient.send(pattern, payload));

    if(authResponse.status === 'success'){
      return {
        message: 'Order created successfully!',
        user: authResponse.user,
      }
    }
    return {
      message: 'Order create failed!',
      error: authResponse.message,
    }
  }
}
```
---


#### `order-service/src/main.ts`
```ts
import { Logger } from '@nestjs/common';
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app/app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  const globalPrefix = 'api';
  app.setGlobalPrefix(globalPrefix);
  const port = process.env.PORT || 5003;
  await app.listen(port);
  Logger.log(
    `🚀 Application is running on: http://localhost:${port}/${globalPrefix}`,
  );
}

bootstrap();
```
---

#### OUTPUT
#### Terminal
```bash
npx nx serve auth-service
```
```bash
npx nx serve order-service
```
---

<img width="1543" height="340" alt="image" src="https://github.com/user-attachments/assets/b05ce1a4-9490-42f8-8ade-a48af27072a8" />


<img width="470" height="249" alt="image" src="https://github.com/user-attachments/assets/a5242f2f-fec7-4011-81b2-e8f8c5d68717" />

<img width="1544" height="182" alt="image" src="https://github.com/user-attachments/assets/f837ae73-fffe-487c-b2c9-dd703e66f1c2" />
