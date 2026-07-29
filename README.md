## NestJS-Architecture

![](https://imgur.com/9BhS7O1.png)
## ১. Module (`app.module.ts`) — সব কিছুর কেন্দ্র

NestJS এর application মূলত **Module** দিয়ে গঠিত। প্রতিটি application এর একটা root module থাকে, যেটাকে বলা হয় `AppModule`।

- Module হলো একটা container, যেখানে related controller আর provider (service) গুলো group করে রাখা হয়।
- `@Module()` decorator দিয়ে বলে দেওয়া হয় — কোন controller, কোন provider, আর কোন অন্য module এখানে ব্যবহার হবে।
- বড় application এ সাধারণত feature অনুযায়ী আলাদা আলাদা module বানানো হয় (যেমন: `UserModule`, `AuthModule`), আর সবগুলোকে `AppModule` এর ভেতর import করা হয়।

```ts
@Module({
  imports: [],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

## ২. Controller (`app.controller.ts`) — Request Handle করে

Controller এর কাজ হলো **incoming HTTP request receive করা এবং response ফেরত পাঠানো**।

<img width="970" height="420" alt="image" src="https://github.com/user-attachments/assets/ce9d77e2-e614-4b63-9b30-65ac55c16815" />


- `@Controller()` decorator দিয়ে route path define করা হয়।
- এর ভেতরে `@Get()`, `@Post()`, `@Put()`, `@Delete()` ইত্যাদি decorator দিয়ে আলাদা আলাদা route handle করা হয়।
- Controller নিজে কোনো business logic রাখে না — logic এর জন্য এটা Service কে call করে।

```ts
@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get()
  getHello(): string {
    return this.appService.getHello();
  }
}
```

## ৩. Service / Provider (`app.service.ts`) — Business Logic এর জায়গা

Service (একে **Provider**-ও বলা হয়) হলো সেই জায়গা, যেখানে আসল business logic, database call, বা calculation লেখা হয়।

- `@Injectable()` decorator দিয়ে mark করা হয়, যাতে NestJS এটাকে **Dependency Injection** এর মাধ্যমে অন্য জায়গায় (যেমন Controller এ) inject করতে পারে।
- এভাবে Controller আর Service আলাদা থাকে — code clean আর testable হয়।

```ts
@Injectable()
export class AppService {
  getHello(): string {
    return 'Hello World!';
  }
}
```

## ৪. Dependency Injection (DI) — NestJS এর মূল শক্তি

Controller এর constructor এ যখন `AppService` নেওয়া হয়েছে, তখন সেটা নিজে থেকে `new AppService()` করে বানাতে হয়নি — NestJS নিজেই এটা inject করে দিয়েছে।

- এটাকেই বলে **Dependency Injection (DI)**।
- এর ফলে module গুলো একে অপরের সাথে **loosely coupled** থাকে, এবং testing করাও সহজ হয়ে যায় (mock service দিয়ে সহজে test করা যায়)।

## ৫. `main.ts` — Application এর Entry Point

এটাই সেই ফাইল, যেখান থেকে application টা শুরু হয় (bootstrap হয়)।

```ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();
```

- `NestFactory.create(AppModule)` — root module কে দিয়ে পুরো application তৈরি করে।
- `app.listen(3000)` — server কে একটা port এ চালু করে।

## ৬. `app.controller.spec.ts` — Testing File

এই ফাইলটা `app.controller.ts` এর জন্য **unit test** লেখার জায়গা। NestJS এ প্রতিটি controller/service এর সাথে সাধারণত এরকম একটা `.spec.ts` ফাইল থাকে, যাতে code এর প্রতিটা অংশ আলাদাভাবে test করা যায়।

## সব মিলিয়ে Request Flow টা এমন

```
Client Request
      ↓
main.ts (server শুরু করে)
      ↓
AppModule (route কে সঠিক Controller এ পাঠায়)
      ↓
Controller (request receive করে)
      ↓
Service (business logic execute করে)
      ↓
Controller (response ফেরত দেয়)
      ↓
Client Response
```

## সংক্ষেপে

| ফাইল | কাজ |
|---|---|
| `main.ts` | Application bootstrap/start করে |
| `app.module.ts` | সব controller ও provider কে organize করে |
| `app.controller.ts` | HTTP request receive করে |
| `app.service.ts` | Business logic রাখে |
| `app.controller.spec.ts` | Unit test লেখার জায়গা |

এই **Module → Controller → Service** pattern টাই NestJS architecture এর মূল ভিত্তি, যা application কে scalable এবং maintainable রাখে।
