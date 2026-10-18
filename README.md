## Heading...

[Nx Monorepo Setup](https://github.com/Omarmdwasimuddin/Nx-Monorepo-Setup)

#### Install Microservices package
```bash
npm i @nestjs/microservices@^11
```
---

#### `auth-service/src/main.ts`
```bash
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
```bash
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


