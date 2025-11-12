# Жизненный цикл ASP.NET Core-приложения
## 1. Короткий TL;DR (1-минутная версия для собеса)

1. Программа создаёт `Host` через `WebApplicationBuilder` (или старый `IHostBuilder`/`IWebHostBuilder`).
    
2. Конфигурация и логирование загружаются, регистрируются сервисы в DI (`ConfigureServices`).
    
3. Контейнер строится; затем конфигурируется конвейер middleware (`Configure` / `Program` после `builder.Build()`), маршруты и endpoints.
    
4. Хост стартует: запускаются `IHostedService`/`BackgroundService` (через `StartAsync`), Kestrel начинает слушать.
    
5. Для каждого HTTP-запроса создаётся `HttpContext` и **Request scope** DI; запрос проходит через middleware pipeline к endpoint и обратно.
    
6. При завершении вызваются `IHostApplicationLifetime` события и `StopAsync` у `IHostedService`, затем контейнер и ресурсы освобождаются.
    

---

## 2. Пошаговый разбор (детально)

### A. Создание билдера / хоста

- В современном .NET (6+), стандартный шаблон:
    

`var builder = WebApplication.CreateBuilder(args); // создаёт HostBuilder + WebHost // builder.Configuration, builder.Logging, builder.Services... var app = builder.Build();                       // строит DI-контейнер и хост // configure middleware + endpoints app.Run();                                       // запускает хост и блокирует текущий поток`

- В старых версиях использовали `Host.CreateDefaultBuilder()` / `WebHost.CreateDefaultBuilder()` + `Startup` с `ConfigureServices` + `Configure`.
    

**Что важно знать:**

- `CreateBuilder`/`CreateDefaultBuilder` инициирует загрузку конфигурации (appsettings, env vars, command line), провайдеров логирования и базовых хостовых настроек.
    
- В этот момент можно настраивать `IHostBuilder` (сервер, окружение, content root).
    

---

### B. Фазы конфигурации и регистрации сервисов

### Host vs App конфигурация

- Есть **host configuration** (для хоста) и **app configuration** (для приложения). Источники конфигурации применяются в определённом порядке: файлы (appsettings.json), environment variables, command line и пр. Позже источники могут переопределять предыдущие.
    

### Регистрация сервисов

- `builder.Services` — регистрируешь все сервисы в DI (`AddSingleton`, `AddScoped`, `AddTransient`, `AddDbContext`, Identity, Auth, etc.).
    
- `ConfigureServices` (в `Startup`) вызывается до построения контейнера.
    
- После регистрации — вызывается `Build()` → **контейнер создаётся** (ServiceProvider).
    

**Ключевой момент:** до `Build()` нельзя разрешать scoped сервисы для запроса — контейнер ещё не собран.

---

### C. Построение middleware pipeline (Configure)

- В классическом `Startup` вызывается `Configure(IApplicationBuilder app, ...)`, где строится pipeline:
    
    - `UseStaticFiles()`, `UseRouting()`, `UseAuthentication()`, `UseAuthorization()` и т.д.
        
- В minimal API — pipeline задаётся после `var app = builder.Build()` через `app.Use...`, `app.Map...`.
    
- **Порядок регистрации важен** — middleware выполняются в порядке регистрации; ответы идут в обратном порядке.
    

**Замечание:** pipeline — это не «код выполняется в Build time»; он формируется в Build, но обработка запросов через него идёт многократно в runtime.

---

### D. Start (app.Run) — запуск хоста

- `app.Run()` стартует web host: Kestrel (или другой сервер) начинает слушать, запускаются зарегистрированные `IHostedService` (их `StartAsync` вызывается).
    
- `IHostApplicationLifetime`/`IHostLifetime` предоставляет события жизненного цикла: `ApplicationStarted`, `ApplicationStopping`, `ApplicationStopped`.
    
- Под капотом стартует цикл прослушивания TCP/HTTP, обрабатываются входящие соединения.
    

---

### E. Обработка HTTP-запроса (runtime)

1. **Kestrel** принимает TCP соединение → парсит HTTP → создает `HttpContext`.
    
2. Для запроса создаётся **Scope** DI (scope per request) — это ключевая гарантия: `AddScoped` сервисы живут в пределах одного запроса.
    
3. `HttpContext` проходит через middleware pipeline:
    
    - Каждый middleware получает `HttpContext` и может вызывать `await next()` для передачи дальше.
        
    - Routing middleware определяет `Endpoint`, EndpointMiddleware вызывает endpoint (контроллер/Minimal API).
        
4. Выполнение endpoint'а (контроллер) — может разрешать сервисы из DI (scoped, transient, singleton).
    
5. После завершения ответа:
    
    - Scope удаляется → `IDisposable` у scoped сервисов вызывается.
        
    - `HttpContext` освобождается.
        

**Важно:** Middleware не должны долго блокировать поток; асинхронность и правильное использование `await` — обязательны.

---

### F. Shutdown (остановка)

- При сигнале завершения (SIGTERM, Ctrl+C, IIS recycle):
    
    1. Хост вызывает `IHostApplicationLifetime.ApplicationStopping`.
        
    2. `IHostedService.StopAsync(cancellationToken)` вызываются для всех сервисов; тут можно корректно завершить BackgroundServices.
        
    3. Завершаются открытые соединения (graceful shutdown, ждём завершения активных запросов).
        
    4. `ApplicationStopped` событие.
        
    5. DI контейнер и disposable сервисы уничтожаются.
        

**Примечание:** для реального production часто настраивают период ожидания graceful shutdown; важно, чтобы `StopAsync` уважал CancellationToken.

---

## 3. Важные детали и собес-ловушки (что нужно уметь объяснять)

### 1) Различие `Build()` и `Run()`

- `Build()` — строит контейнер и pipeline (готовит приложение), но не запускает прослушивание.
    
- `Run()` — запускает хост и блокирует поток (если не в background).
    

### 2) Когда вызывается `ConfigureServices` и `Configure`

- `ConfigureServices` — во время сборки host; регистрируешь зависимости.
    
- `Configure` / вызов `app.Use...` — строит middleware pipeline после `Build()` (в minimal API код обычно идёт после `builder.Build()`).
    

### 3) Scope per request

- `Scoped` сервисы создаются **один раз на HTTP-запрос** (в web host), и удаляются после завершения.
    
- Частая ошибка — внедрять `Scoped` в `Singleton` напрямую (нельзя: будет захвачен scope с lifetime того singleton).
    

### 4) Порядок middleware

- `UseRouting()` должен быть до `UseAuthorization()`. Если порядок нарушен — authorization не увидит route data/endpoint.
    
- `ExceptionHandling` middleware должен быть самым ранним (до других middleware), но `DeveloperExceptionPage` только в dev.
    

### 5) Где вызывать DB миграции / инициализацию?

- Часто делают в начале `Program.cs` перед `app.Run()`:
    

`using (var scope = app.Services.CreateScope()) {     var db = scope.ServiceProvider.GetRequiredService<MyDbContext>();     db.Database.Migrate(); }`

- Важно: делать это синхронно и безопасно.
    

### 6) Background tasks

- Реализация через `IHostedService` или `BackgroundService`. `StartAsync` вызывается при старте, `StopAsync` — при остановке. Respect CancellationToken!
    

### 7) Exception handling и завершение запросов

- Некорректный middleware, который ловит исключения, но не возвращает корректный ответ, может «отключить» pipeline. Нужно корректно логировать и формировать ответ.
    

### 8) Как масштабируется приложение (Server GC)

- Различия Workstation vs Server GC: серверный режим создаёт несколько heap сегментов для высокой нагрузки; влияет на масштабирование и latency.
    

---

## 4. Полезные фрагменты кода (минимал/Startup)

### Minimal (.NET 6+)

`var builder = WebApplication.CreateBuilder(args); builder.Services.AddControllers(); builder.Services.AddDbContext<MyDbContext>(...);  var app = builder.Build();  if (app.Environment.IsDevelopment())     app.UseDeveloperExceptionPage();  app.UseHttpsRedirection(); app.UseRouting(); app.UseAuthentication(); app.UseAuthorization();  app.MapControllers();  app.Run();`

### Классический `Startup`

`public class Startup {     public void ConfigureServices(IServiceCollection services)     {         services.AddControllers();     }      public void Configure(IApplicationBuilder app, IWebHostEnvironment env)     {         app.UseRouting();         app.UseAuthorization();         app.UseEndpoints(endpoints => endpoints.MapControllers());     } }`

---

## 5. Что реально хорошенько проговорить на собесе (чеклист — тезисы для устного ответа)

- Последовательность: **CreateBuilder → ConfigureServices → Build → Configure (middleware) → Run**.
    
- В чём разница между `Build()` и `Run()`; когда контейнер создаётся.
    
- Scope per request: `AddScoped` → один экземпляр на запрос; почему нельзя внедрять scoped в singleton.
    
- Middleware pipeline: `Use`, `Run`, `Map`, порядок важен; routing/auth placement.
    
- `IHostedService` / `BackgroundService`: когда стартуют/остановливаются.
    
- Graceful shutdown: `StopAsync`, `CancellationToken` важно уважать.
    
- Где безопасно выполнять DB migrations / seeding (CreateScope перед Run).
    
- Конфигурация: порядок провайдеров (appsettings → env vars → command line).
    
- Разница Generic Host vs Web Host (кратко: сейчас рекомендован Generic Host, он унифицирован).
    

---

##3 6. Часто задаваемые вопросы на интервью (формулировки + краткие ответы)

Q: _Когда создаётся DI контейнер?_  
A: При вызове `Build()`; до этого ты только настраиваешь `IServiceCollection`.

Q: _Почему `Scoped` нельзя инъектить в `Singleton`?_  
A: Потому что singleton живёт дольше; scoped зависимость будет захвачена единожды при создании singleton и фактически станет singleton'ом (нарушение ожиданий). Решение — `IServiceProvider.CreateScope()` или фабрика `IServiceScopeFactory`.

Q: _Где поставить `UseExceptionHandler`?_  
A: Ранним в pipeline, до middleware, где возможно должны быть обработаны исключения (но после middleware, которые должны перехватывать ошибки сами).

Q: _Как корректно завершить background task при shutdown?_  
A: Реализовать `BackgroundService`, в `ExecuteAsync` следить за `stoppingToken`, в `StopAsync` корректно закрывать ресурсы.

---

## 7. Короткая шпаргалка для проговаривания (30–60 секунд)

«ASP.NET Core приложение строится через `WebApplicationBuilder`/`HostBuilder`. Сначала загружается конфигурация и логгинг, затем регистрируются сервисы (`ConfigureServices`), потом вызывается `Build()` — контейнер создаётся. После `Build()` формируется middleware-pipeline (`Configure`/`app.Use...`). `Run()` запускает хост: Kestrel начинает слушать и вызываются `IHostedService.StartAsync`. На каждый HTTP-запрос создаётся `HttpContext` и request-scope DI; запрос проходит через middleware к endpoint и обратно. При завершении вызываются `StopAsync` у hosted services и события `ApplicationStopping`/`ApplicationStopped`; контейнер и disposable сервисы освобождаются.»


