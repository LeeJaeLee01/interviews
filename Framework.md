## Câu 1: Clean Architecture

Clean Architecture là một phong cách kiến trúc phần mềm do Robert C. Martin (Uncle Bob) phổ biến, với mục tiêu chính là:

(Clean Architecture is a software architecture style introduced by Robert C. Martin with the main goal of:)

- **Tách biệt rõ ràng giữa business logic (domain)** và các chi tiết hạ tầng (database, UI, framework, message broker…).

(**Clearly separate the business logic**)

- **Giảm phụ thuộc**, giúp hệ thống:
  - Dễ thay đổi giao diện (UI, API) mà không làm vỡ lõi nghiệp vụ.
  - Dễ thay đổi database / framework / thư viện bên ngoài.
  - Dễ viết unit test cho domain thuần.

### Nguyên tắc cốt lõi

- **Độc lập framework**: Framework (Spring, .NET, NestJS, Django, v.v.) chỉ là “plugin” vào hệ thống, không được chi phối thiết kế domain.
- **Độc lập UI**: Có thể thay đổi giao diện (Web, Mobile, CLI…) mà không ảnh hưởng business rules.
- **Độc lập database**: Domain không phụ thuộc vào loại DB (SQL, NoSQL, in-memory…).
- **Độc lập với external systems**: Message queue, cache, 3rd‑party services chỉ là chi tiết triển khai.
- **Phụ thuộc một chiều (Dependency Rule)**: Mũi tên phụ thuộc luôn hướng **từ ngoài vào trong** (outer layers → inner layers). Tầng trong không biết gì về tầng ngoài.

### Các tầng chính

Tên gọi có thể khác nhau tùy tài liệu/proejct, nhưng ý tưởng chung:

- **Entities / Domain Model (Core Domain)**
  - Chứa: domain entities, value objects, domain services, business rules thuần túy.
  - Không phụ thuộc framework, không phụ thuộc IO.
  - Ví dụ: `Order`, `Customer`, `Money`, `OrderDomainService`.
- **Use Cases / Application Layer**
  - Chứa: application services / use case services điều phối luồng nghiệp vụ cho từng ca sử dụng cụ thể.
  - Phụ thuộc vào Domain, nhưng không phụ thuộc vào UI, DB cụ thể.
  - Giao tiếp với tầng ngoài qua **interfaces/ports** (ví dụ: `OrderRepository`, `PaymentGateway`).
  - Ví dụ: `PlaceOrderUseCase`, `CancelOrderUseCase`.
- **Interface Adapters (Adapters Layer)**
  - Chứa: controller, presenter, view model, mappers, repository implementation.
  - Nhiệm vụ: chuyển đổi dữ liệu giữa format thuận tiện cho framework/UI/DB và format của Domain / Use Cases.
  - Ví dụ:
    - HTTP controller map từ HTTP request → DTO → Use Case input.
    - Repository implementation map từ ORM model → Domain entity.
- **Infrastructure / Framework & Drivers**
  - Chứa: code cụ thể cho database (ORM, SQL scripts), queues, HTTP clients, file system, framework bootstrapping.
  - Implement các interface (ports) được định nghĩa ở Application/Domain.
  - Ví dụ: `JpaOrderRepository`, `MongoCustomerRepository`, `StripePaymentGateway`, `KafkaEventPublisher`.

### Dependency Rule (quy tắc phụ thuộc)

- Tất cả phụ thuộc phải hướng **từ ngoài vào trong**:
  - Infrastructure → Interface Adapters → Application → Domain.
- Tầng trong **không được**:
  - Import class từ tầng ngoài.
  - Biết về framework (Spring, Nest, Django, Rails…).
  - Biết về chi tiết DB, HTTP, UI.
- Để “ngược chiều” dependency (UI gọi vào Use Case, Use Case gọi repo implement ở infra) mà vẫn tuân thủ rule, ta dùng:
  - **Inversion of Control (IoC)**.
  - **Dependency Inversion Principle (DIP)**: inner layer define `interface`, outer layer implement và inject.

### Mối quan hệ với Ports & Adapters / Hexagonal

- Clean Architecture thường được triển khai cùng với:
  - **Hexagonal Architecture (Ports & Adapters)**.
  - **Onion Architecture**.
- Ý tưởng chung:
  - **Port**: interface do domain/application expose ra (ví dụ: `OrderRepository`, `PaymentGateway`).
  - **Adapter**: implementation cụ thể cho từng công nghệ (SQL, REST, Kafka…).
  - Điều này cho phép thay đổi adapter mà không chạm vào core.

### Ưu điểm

- **Tính testable cao**:
  - Domain và Use Cases là pure code, dễ viết unit test, không cần boot framework, không cần DB thật.
- **Tính maintainable và evolvable cao**:
  - Dễ mở rộng tính năng > tách biệt responsibility rõ ràng.
  - Dễ refactor UI, DB, frameworks.
- **Giảm coupling với công nghệ**:
  - Dễ migrate từ một framework/DB sang thứ khác.
- **Tập trung vào domain**:
  - Thiết kế xoay quanh nghiệp vụ (domain‑driven), chứ không bị “lead” bởi framework.

### Nhược điểm và trade-off

- **Độ phức tạp thiết kế ban đầu cao**:
  - Nhiều tầng, nhiều interface, dễ bị “over-engineering” cho dự án nhỏ.
- **Chi phí onboarding**:
  - Dev mới cần thời gian hiểu layers, flow, conventions.
- **Nhiều boilerplate**:
  - DTO, mapper, adapter, interface… nếu không được codegen hoặc dùng patterns hợp lý sẽ khá tốn công.

### Khi nào nên dùng Clean Architecture

- **Phù hợp**:
  - Hệ thống vừa và lớn, nhiều nghiệp vụ cốt lõi, vòng đời dài.
  - Khả năng phải thay đổi UI/DB/framework trong tương lai là cao.
  - Tổ chức coi trọng testability, maintainability, code quality.
- **Không nên quá “heavy”**:
  - Prototype, MVP rất nhỏ, vòng đời ngắn → có thể áp dụng chọn lọc (ví dụ tách domain + application đơn giản, không cần đủ 4 tầng).

### Guidelines áp dụng thực tế

- **Bắt đầu từ Domain & Use Cases**:
  - Viết domain model, business rules, rồi mới nghĩ đến hạ tầng.
- **Define interfaces ở trong, implement ở ngoài**:
  - Ví dụ: `OrderRepository` định nghĩa trong Application, `SqlOrderRepository` implement ở Infrastructure.
- **Giữ domain thuần**:
  - Không annotation framework, không logic IO, không access static từ framework.
- **Áp dụng dần dần**:
  - Không cần phải “full Clean” ngay từ đầu.
  - Có thể refactor từng module/slice sang cấu trúc Clean khi chúng trở nên phức tạp.

### Tóm tắt

- **Clean Architecture** giúp:
  - Tách biệt rõ core business với hạ tầng.
  - Giảm coupling với framework/DB.
  - Tăng testability & maintainability.
- **Chi phí**:
  - Thiết kế và cấu trúc ban đầu phức tạp hơn.
  - Cần discipline của cả team để giữ boundaries rõ ràng.

---

## Câu 2: Domain-Driven Design (DDD)

Domain-Driven Design (Eric Evans) là một cách tiếp cận thiết kế phần mềm **lấy domain nghiệp vụ làm trung tâm**, nhấn mạnh việc:

- **Hiểu sâu domain** thông qua trao đổi chặt chẽ với domain experts.
- **Mô hình hóa domain** thành code một cách rõ ràng, nhất quán, và dễ thay đổi.
- **Sử dụng ngôn ngữ chung (Ubiquitous Language)** giữa business và dev để giảm hiểu lầm.

DDD rất phù hợp để kết hợp với Clean Architecture: Clean tập trung vào **cấu trúc tầng**, DDD tập trung vào **mô hình domain và boundaries**.

### Khái niệm cốt lõi

- **Ubiquitous Language**:
  - Ngôn ngữ chung được cả business và dev sử dụng trong mọi tài liệu, cuộc họp, code.
  - Tên class, method, event, field nên trùng với ngôn ngữ business (tránh technical jargon không cần thiết).

- **Entity**:
  - Đối tượng có **identity** (định danh) ổn định theo thời gian, dù thuộc tính có thể thay đổi.
  - Ví dụ: `Customer`, `Order`, `Account`.

- **Value Object**:
  - Đối tượng **không có identity riêng**, được xác định bởi tập thuộc tính; thường là immutable.
  - Ví dụ: `Money`, `Address`, `DateRange`.

- **Aggregate & Aggregate Root**:
  - **Aggregate**: cụm liên quan của Entities/Value Objects tạo thành một unit về consistency.
  - **Aggregate Root**: entry point duy nhất để thao tác aggregate, quản lý invariants.
  - Ví dụ: `Order` là Aggregate Root, chứa `OrderLine`, `ShippingAddress` (VO).

- **Domain Service**:
  - Chứa business logic **không thuộc riêng về một Entity/Value Object** nào.
  - Ví dụ: tính phí giao hàng, chính sách discount phức tạp.

- **Repository**:
  - Abstraction cho persistence, cung cấp cảm giác “làm việc với collection in-memory” của Aggregate Root.
  - Ví dụ: `OrderRepository`, `CustomerRepository`.

### Strategic Design

Strategic DDD tập trung vào bức tranh lớn của hệ thống thông qua:

- **Bounded Context**:
  - “Biên giới” nơi một mô hình domain và Ubiquitous Language cụ thể có hiệu lực.
  - Cùng một từ có thể mang ý nghĩa khác ở các bounded context khác nhau.
  - Ví dụ:
    - `Billing` context: “Customer” chứa thông tin thanh toán, công nợ.
    - `CRM` context: “Customer” chứa thông tin marketing, tương tác.

- **Context Mapping**:
  - Xác định mối quan hệ giữa các bounded context:
    - **Customer/Supplier**, **Conformist**, **Anti-Corruption Layer (ACL)**, **Shared Kernel**, **Separate Ways**...
  - Giúp kiểm soát coupling giữa các team/hệ thống.

### Tactical Design

Tactical DDD tập trung vào các building blocks ở mức code:

- Entities, Value Objects, Aggregates, Domain Services, Repositories.
- Domain Events: mô tả các sự kiện có ý nghĩa business (ví dụ: `OrderPlaced`, `PaymentFailed`).
- Application Services / Use Cases: điều phối gọi domain, repository, event.

Khi kết hợp với Clean Architecture:

- Domain Layer của Clean chính là nơi đặt **Entities, Value Objects, Domain Services, Domain Events**.
- Application Layer chứa **Use Cases / Application Services**, Repository interfaces.

### DDD và kiến trúc

- **DDD không phải một kiến trúc cụ thể**, mà là:
  - Một set patterns + mindset để thiết kế hệ thống xoay quanh domain.
- DDD thường được triển khai tốt nhất với:
  - **Layered Architecture**, **Hexagonal Architecture**, **Clean Architecture**, **Onion Architecture**…
- DDD giúp:
  - Xác định **Bounded Context**, từ đó map ra modules/microservices.
  - Thiết kế model domain rõ ràng, tách biệt core domain, supporting domain, generic subdomain.

### Khi nào nên áp dụng DDD

- **Phù hợp**:
  - Hệ thống phức tạp về nghiệp vụ (business complexity cao hơn technical complexity).
  - Nhiều domain experts, nhiều team, nhiều subdomain khác nhau.
  - Sản phẩm có vòng đời dài, liên tục thay đổi theo business.
- **Ít phù hợp**:
  - CRUD app đơn giản, logic mỏng, vòng đời ngắn.
  - Hệ thống mà business gần như không đổi hoặc rất nhỏ.

### Ưu điểm

- **Mô hình domain rõ ràng, sát business**:
  - Giảm gap giữa yêu cầu và implementation.
- **Dễ thay đổi theo business**:
  - Thay đổi domain model trở nên tự nhiên, ít vỡ hệ thống.
- **Tổ chức theo domain chứ không theo kỹ thuật**:
  - Team align theo bounded context, giảm phụ thuộc chéo.

### Nhược điểm và thách thức

- **Đòi hỏi collaboration cao**:
  - Cần thời gian và effort để domain experts và dev làm việc chung, refine Ubiquitous Language.
- **Đường cong học tập cao**:
  - Patterns, terminology của DDD khá nhiều, dễ gây “bội thực” nếu áp dụng máy móc.
- **Rủi ro over-engineering**:
  - Áp dụng full DDD cho hệ thống đơn giản sẽ làm mọi thứ rườm rà, khó ship nhanh.

### Tóm tắt (DDD + Clean Architecture)

- **DDD** tập trung vào:
  - Mô hình hóa domain, xác định bounded context, sử dụng Ubiquitous Language.
  - Xây domain model với Entities, Value Objects, Aggregates, Domain Services, Events.
- **Clean Architecture** tập trung vào:
  - Tách layers, kiểm soát dependencies từ ngoài vào trong.
- **Kết hợp**:
  - Domain Layer của Clean dùng tactical DDD.
  - Strategic DDD (Bounded Context, Context Map) định hình cấu trúc module/microservice tổng thể.

---

## Câu 3: Các câu hỏi phỏng vấn về NestJS

### Cơ bản

- **NestJS là gì?** So sánh NestJS với Express, Koa.  
  - **Trả lời**: NestJS là một framework Node.js xây trên Express (hoặc Fastify), sử dụng TypeScript và các pattern như DI, decorator để xây dựng server-side app có cấu trúc module rõ ràng. So với Express/Koa (chỉ là web framework “mỏng”), NestJS cung cấp kiến trúc opinionated, tích hợp testing, DI, modularization, giúp dự án lớn dễ maintain hơn.

- **Kiến trúc của NestJS** dựa trên những khái niệm/lý thuyết nào? (OOP, FP, MVC, DI, Decorator, Module…)  
  - **Trả lời**: NestJS kết hợp OOP, FP và FP reactive (RxJS) với các pattern như Dependency Injection, module hóa, decorators, và kiến trúc layer (Controller → Service → Repository). Nó chịu ảnh hưởng mạnh từ kiến trúc của Angular (module, provider, decorator).

- **Module trong NestJS là gì?** Tại sao mọi thứ đều xoay quanh module? Cách tổ chức modules trong một ứng dụng lớn.  
  - **Trả lời**: Module (`@Module`) là đơn vị tổ chức code logic, gom nhóm controller, provider, export/import giữa các phần của hệ thống. Ứng dụng lớn thường được chia thành nhiều feature module theo domain (UserModule, AuthModule, OrderModule…), giúp tách biệt trách nhiệm và dễ tái sử dụng.

- **Controller là gì?** Decorator `@Controller()`, `@Get()`, `@Post()`, `@Param()`, `@Body()`, `@Query()` dùng để làm gì?  
  - **Trả lời**: Controller xử lý HTTP request/response, map route vào handler tương ứng. `@Controller` định nghĩa prefix, các decorator method như `@Get`, `@Post` map HTTP verb + path, còn `@Param`, `@Body`, `@Query` giúp inject dữ liệu từ URL, body, query vào tham số hàm.

- **Provider là gì?** Sự khác nhau giữa Controller và Provider.  
  - **Trả lời**: Provider là các class có thể được NestJS quản lý và inject (service, repository, factory…). Controller là entry point cho request, còn provider chứa business logic và được controller/service khác gọi thông qua DI. Provider không trực tiếp xử lý HTTP routing.

- **Dependency Injection trong NestJS hoạt động như thế nào?** Các scope của provider (default, `Request`, `Transient`).  
  - **Trả lời**: NestJS duy trì một IoC container, tự tạo instance và inject dependencies dựa vào metadata (constructor type, `@Inject`). Provider mặc định là singleton (scope default), có thể cấu hình `scope: Scope.REQUEST` để mỗi request có một instance, hoặc `Scope.TRANSIENT` để mỗi injection tạo mới.

- **Lifecycle của một HTTP request** đi qua NestJS như thế nào? (Middleware → Guard → Interceptor → Pipe → Controller → Service → …)  
  - **Trả lời**: Request đi qua middleware (tiền xử lý chung), sau đó tới guards (quyết định cho phép hay chặn), tiếp đến pipes để validate/transform input, rồi interceptor để trước/sau xử lý thêm (logging, mapping response), cuối cùng controller handler gọi service; nếu có exception filter thì bắt và format lỗi trả về.

- **Pipe là gì?** Phân biệt `ValidationPipe`, `ParseIntPipe`, `ParseUUIDPipe`, pipe tự custom.  
  - **Trả lời**: Pipe dùng để transform và/hoặc validate dữ liệu đầu vào trước khi vào handler. `ValidationPipe` dùng `class-validator` để validate DTO, các pipe như `ParseIntPipe`, `ParseUUIDPipe` parse và validate kiểu dữ liệu; custom pipe dùng khi cần logic transform/validate đặc thù.

- **Guard là gì?** Sử dụng `CanActivate`, `AuthGuard`, `RolesGuard` như thế nào?  
  - **Trả lời**: Guard quyết định request có được phép vào handler hay không (authorization, auth). Implement `CanActivate` để kiểm tra điều kiện, `AuthGuard` thường check JWT/session, `RolesGuard` check role dựa trên metadata (`@Roles`) và user trong request.

- **Interceptor là gì?** Các use case thường gặp (logging, transform response, caching…).  
  - **Trả lời**: Interceptor “bọc” quanh execution context, cho phép chạy logic trước và sau handler. Use case: logging thời gian xử lý, mapping dữ liệu domain → DTO response, áp dụng caching, hoặc uniform response format.

- **Exception Filter là gì?** Cách custom global exception filter (`@Catch()`, `HttpExceptionFilter`).  
  - **Trả lời**: Exception filter bắt exception và chuẩn hóa response trả về client. Custom filter dùng `@Catch()` để bắt loại lỗi cụ thể, override `catch()` để map sang HTTP status & body; có thể đăng ký global để toàn app dùng chung.

- **Middleware trong NestJS hoạt động ra sao?** Khác gì với Guard/Interceptor?  
  - **Trả lời**: Middleware chạy trước khi request vào NestJS route handler, thường dùng cho logging cơ bản, body parsing, attach thông tin vào request. Khác với guard/interceptor (chạy trong context NestJS, có access metadata), middleware giống middleware Express, không aware metadata route.

- **ConfigModule** dùng để làm gì? Cách load config từ `.env` và validate bằng `Joi` hoặc `zod`.  
  - **Trả lời**: `ConfigModule` cung cấp cơ chế quản lý cấu hình, load biến môi trường và inject `ConfigService` vào provider. Có thể dùng `ValidationSchema` (Joi/zod) để đảm bảo các biến môi trường hợp lệ khi app khởi động.

### Làm việc với dữ liệu & HTTP

- **Cách tích hợp ORM với NestJS**: TypeORM, Prisma, Mongoose… Ưu/nhược từng cách.  
  - **Trả lời**: NestJS có module chính thức cho TypeORM, Mongoose và tích hợp tốt với Prisma. TypeORM hỗ trợ active record & data mapper, dễ dùng nhưng khá “nặng”; Prisma có DX tốt, schema-first, mạnh cho type-safety; Mongoose phù hợp MongoDB, flexible nhưng kém type-safety hơn nếu không cấu hình kỹ.

- **Repository Pattern trong NestJS + TypeORM**: `@InjectRepository()`, `Repository`, `DataSource`.  
  - **Trả lời**: Với TypeORM, ta inject repository vào service bằng `@InjectRepository(Entity)`, từ đó dùng `Repository` để CRUD. `DataSource` đại diện connection, cho phép lấy custom repository hoặc chạy query builder/transaction nâng cao.

- **Transaction** trong NestJS + TypeORM hoặc Prisma được xử lý như thế nào?  
  - **Trả lời**: Với TypeORM dùng `dataSource.transaction()` hoặc `QueryRunner` để wrap nhiều thao tác DB trong một transaction. Với Prisma dùng `prisma.$transaction()` để chạy nhiều lệnh logic atomic; có thể inject service/repo vào trong callback để giữ boundary rõ ràng.

- **Validation với `class-validator` và `class-transformer`**: Cách dùng DTO, `@IsString()`, `@IsOptional()`, `@ValidateNested()`…  
  - **Trả lời**: Ta định nghĩa DTO class với decorator `class-validator` rồi bật `ValidationPipe` (thường global). `class-transformer` giúp transform plain object từ request thành instance DTO, còn các decorator như `@IsString`, `@IsOptional`, `@ValidateNested` mô tả rule validate trường tương ứng.

- **Cách cấu hình Global Pipes, Global Filters, Global Guards**.  
  - **Trả lời**: Trong `main.ts`, dùng `app.useGlobalPipes()`, `app.useGlobalFilters()`, `app.useGlobalGuards()` để đăng ký. Với DI, nên dùng `app.useGlobalPipes(new ValidationPipe(...))` hoặc provide qua module và sử dụng token system của Nest.

- **Xử lý file upload** với `@UseInterceptors(FileInterceptor...)` và Multer.  
  - **Trả lời**: Dùng `@UseInterceptors(FileInterceptor('fieldName'))` trên handler, NestJS tích hợp Multer để parse file từ multipart/form-data, tham số `@UploadedFile()` lấy file, có thể cấu hình storage, file filter, size limit.

- **Xử lý pagination, sorting, filtering** trong REST API NestJS.  
  - **Trả lời**: Thường dùng DTO cho query (`page`, `limit`, `sortBy`, `order`, filter fields), validate bằng pipes rồi map sang query builder của ORM. Best practice là trả về cấu trúc có `items`, `total`, `page`, `limit` để client dễ xử lý.

### Auth, Security

- **Cơ chế authentication trong NestJS** với `@nestjs/passport` và `@nestjs/jwt`.  
  - **Trả lời**: NestJS dùng Passport để plug các strategy (local, JWT, OAuth2…). `@nestjs/jwt` cung cấp `JwtService` để sign/verify token, `PassportStrategy(JwtStrategy)` dùng để đọc JWT từ header và attach payload vào request.

- **Cách implement JWT Auth**: `AuthModule`, `AuthService`, `JwtStrategy`, `LocalStrategy`.  
  - **Trả lời**: Thường tạo `AuthModule` chứa `AuthService` để validate user, tạo token, `LocalStrategy` xử lý login bằng username/password, `JwtStrategy` verify token cho các route protected. Controller dùng `AuthGuard('local')` cho login và `AuthGuard('jwt')` cho các route cần bảo vệ.

- **Role-based access control (RBAC)**: sử dụng `RolesGuard`, decorator `@Roles()`.  
  - **Trả lời**: Tạo decorator `@Roles(...roles)` set metadata cho handler, `RolesGuard` đọc metadata đó và so sánh với roles trong user (từ JWT hoặc DB). Gắn `@UseGuards(RolesGuard)` hoặc global guard để enforce RBAC thống nhất.

- **Best practices bảo mật** trong NestJS: Helmet, rate limiting, CORS, CSRF (nếu có), bảo vệ sensitive config.  
  - **Trả lời**: Bật `helmet` để set HTTP headers, dùng rate limiting (như `@nestjs/throttler`), cấu hình CORS đúng domain, không log plaintext secrets, dùng `ConfigModule` + secrets manager. Nếu dùng session/cookie thì cân nhắc CSRF protection, secure cookie flags.

- **Xử lý refresh token** trong NestJS như thế nào?  
  - **Trả lời**: Tách access token ngắn hạn và refresh token dài hạn; lưu refresh token (hash) trong DB hoặc secure storage, cấp API để rotate/làm mới access token. Khi nhận refresh token, verify, check revoked/rotation, nếu hợp lệ thì phát cặp token mới.

### Microservices & Messaging

- **NestJS Microservices** là gì? Hỗ trợ những transport nào? (TCP, Redis, NATS, Kafka, gRPC, RabbitMQ…).  
  - **Trả lời**: NestJS Microservices là mô hình xây app theo message-based transport thay vì thuần HTTP. Nó support nhiều transport như TCP, Redis, NATS, MQTT, Kafka, RabbitMQ, gRPC… thông qua adapter tương ứng.

- **Cách tạo một microservice trong NestJS**: `NestFactory.createMicroservice`, `ClientsModule`, `@MessagePattern()`.  
  - **Trả lời**: Dùng `NestFactory.createMicroservice()` với config transport để tạo server microservice, trong handler dùng `@MessagePattern(pattern)` để nhận message. Ở phía client, đăng ký `ClientsModule` và inject `ClientProxy` để `send`/`emit` message.

- **Chữ ký message và pattern** trong microservices: `@MessagePattern('pattern_name')`.  
  - **Trả lời**: `@MessagePattern()` xác định pattern (subject/topic/route) mà handler sẽ lắng nghe, tuỳ theo transport (Kafka topic, Redis channel…). Message gồm pattern + payload, server map đến handler tương ứng.

- **So sánh REST vs gRPC trong NestJS**: use case, ưu/nhược, performance.  
  - **Trả lời**: REST đơn giản, dễ debug, tương thích rộng; gRPC dùng HTTP/2 + Protobuf cho performance và type-safety tốt hơn, phù hợp internal microservices. Với NestJS, REST dễ triển khai hơn, gRPC tốt khi cần low latency, strongly-typed contract.

- **Sử dụng EventEmitter hoặc CQRS** cho event-driven trong NestJS như thế nào?  
  - **Trả lời**: `@nestjs/event-emitter` cho phép publish/subscribe event trong nội bộ process; `@nestjs/cqrs` cung cấp infrastructure cho Command, Query, Event, Handler tách biệt. Trong DDD, thường dùng CQRS module để model hoá command handler và domain event rõ ràng.

### WebSocket & Realtime

- **Cách dùng WebSocket Gateway trong NestJS**: `@WebSocketGateway()`, `@SubscribeMessage()`.  
  - **Trả lời**: Tạo class gateway với `@WebSocketGateway()`, mỗi message handler dùng `@SubscribeMessage('event')` để nhận event từ client. Inject service vào gateway để xử lý business logic rồi `client.emit()` hoặc `server.emit()` để broadcast.

- **Cơ chế kết nối giữa client và NestJS WebSocket** (Socket.IO, ws).  
  - **Trả lời**: Mặc định NestJS WebSocket gateway dùng Socket.IO, client kết nối qua URL và namespace cấu hình. Có thể đổi adapter sang ws hoặc các implementation khác, tham số handler nhận `client` và `payload` để thao tác connection.

- **Quản lý room/channel, broadcast event** trong NestJS WebSocket.  
  - **Trả lời**: Với Socket.IO adapter, có thể dùng `client.join(room)` để tham gia room, `server.to(room).emit(...)` để broadcast. Thường lưu userId ↔ socketId/room mapping trong service để gửi message đúng đối tượng.

### Testing & Maintainability

- **Testing trong NestJS**: sử dụng `@nestjs/testing`, `Test.createTestingModule`.  
  - **Trả lời**: Nest cung cấp `TestingModule` để tạo module test, mock dependencies và lấy instance controller/service. Dùng Jest làm test runner, dễ viết unit & integration test.

- **Unit test cho Controller và Service**: mock dependencies bằng `useValue`, `useFactory`.  
  - **Trả lời**: Trong test module, dùng `providers: [{ provide: XService, useValue: mockXService }]` để mock; với factory phức tạp dùng `useFactory`. Unit test controller tập trung vào mapping request → service call, service test tập trung vào business logic.

- **Integration test / e2e test** với `SuperTest` + NestJS.  
  - **Trả lời**: Dùng `NestFactory` tạo app thực tế trong test, rồi dùng `request(app.getHttpServer())` của SuperTest để gọi HTTP endpoint. E2E test verify full stack (middleware, guard, pipe, controller, service, DB giả/real).

- **Tổ chức cấu trúc folder** cho project NestJS lớn: theo domain, theo bounded context, theo module.  
  - **Trả lời**: Best practice là tổ chức theo domain/bounded context (user, auth, order…), mỗi context có module riêng, bên trong chia nhỏ controller/service/repository. Với DDD/Clean, có thể thêm tách domain/application/infrastructure trong mỗi context.

- **Cách áp dụng Clean Architecture/DDD trong NestJS**: tách domain, application, infrastructure layers.  
  - **Trả lời**: Đặt domain model và domain service trong layer domain (không phụ thuộc Nest), application service/use case ở layer application, adapters (controller, repository implementation) ở infrastructure. Module NestJS chủ yếu wire các layer qua DI, đảm bảo dependency luôn từ ngoài vào trong.

### Performance & Deployment

- **Tối ưu performance cho NestJS**: caching với `CacheModule`, `Redis`, gzip, HTTP compression, keep-alive, connection pooling DB.  
  - **Trả lời**: Bật `CacheModule` (memory/Redis) cho các endpoint đọc nhiều, dùng HTTP compression/gzip, keep-alive, và cấu hình connection pooling của DB driver/ORM. Ngoài ra cần tối ưu query, index DB, và tránh logic block event loop.

- **Logging trong NestJS**: `Logger`, custom logger, tích hợp với tools như Winston, Pino.  
  - **Trả lời**: Dùng `Logger` built-in hoặc inject custom logger bằng `useClass` để gửi log tới Winston/Pino. Có thể dùng interceptor/middleware để log request/response chuẩn hóa, kèm correlation id.

- **Cách cấu hình `Cluster` hoặc scale NestJS** phía sau load balancer.  
  - **Trả lời**: Có thể dùng Node.js cluster (hoặc PM2) để chạy nhiều worker trên cùng máy, hoặc scale nhiều container/pod phía sau load balancer (NGINX, ALB…). Cần đảm bảo session/state không lưu trong memory local mà dùng cache/db chung.

- **Triển khai NestJS**: Docker, Kubernetes, serverless (AWS Lambda, GCP Functions)…  
  - **Trả lời**: Đóng gói app bằng Docker (multi-stage build) rồi deploy lên K8s/VM, hoặc dùng `@nestjs/serverless` adapter (hoặc Express adapter) để chạy trên Lambda/Cloud Functions. Tuỳ môi trường mà tinh chỉnh cold start, logging, metrics.

### Các câu hỏi tình huống (scenario-based)

- Bạn sẽ **thiết kế module auth** trong NestJS như thế nào để dễ mở rộng social login (Google, Facebook, Apple…)?  
  - **Trả lời**: Tách `AuthModule` với interface rõ ràng cho các identity provider, mỗi social login là một strategy (Passport strategy) riêng, map về một user model thống nhất. Lưu liên kết giữa user và provider (providerId) để dễ mở rộng thêm provider mới.

- Làm sao để **tái sử dụng logic validation & transformation** giữa nhiều endpoints khác nhau?  
  - **Trả lời**: Đưa logic vào DTO + custom pipes hoặc interceptor chung, đăng ký global hoặc module level. Ngoài ra có thể tách shared service/helper, tránh lặp lại logic trong từng controller.

- Nếu một service cần gọi tới nhiều microservices khác nhau, bạn sẽ **tổ chức code và handle retry/circuit breaker** ra sao?  
  - **Trả lời**: Tạo layer client (service) riêng cho từng microservice, controller/use case chỉ gọi qua các client này. Trong client, wrap call bằng retry/backoff, circuit breaker (lib như `opossum`), và chuẩn hoá error để tránh logic lỗi lan sang domain.

- Khi codebase NestJS ngày càng lớn, bạn sẽ **refactor để tách thành nhiều bounded context/modules** như thế nào?  
  - **Trả lời**: Đầu tiên identify bounded context theo domain, tách module theo context, cô lập dependency (chỉ expose public interface). Khi boundary rõ, có thể tách thành package nội bộ hoặc microservice riêng nếu cần.

- Bạn sẽ **log và trace một request xuyên qua nhiều service NestJS** (distributed tracing) như thế nào?  
  - **Trả lời**: Sử dụng correlation id (trace id) gắn vào request ở entry point, propagate qua header/message tới các service xuống dưới. Tích hợp với OpenTelemetry hoặc APM (Jaeger, Zipkin, Datadog…) để thu thập trace spans xuyên qua các service.

---

## Câu 4: Dependency Injection (DI) là gì?

### Khái niệm

- **Dependency Injection (DI)** là một kỹ thuật/pattern trong đó **thay vì class tự tạo ra các phụ thuộc (dependencies) của mình**, chúng được **“tiêm” (inject)** từ bên ngoài vào (thông qua constructor, setter, hoặc method).
- DI là một cách hiện thực hoá **Inversion of Control (IoC)**: thay vì object tự control việc khởi tạo dependencies, một IoC container/framework sẽ control và cung cấp chúng.

### Ví dụ trực giác

- Không dùng DI:
  - `OrderService` tự `new OrderRepository()` bên trong → `OrderService` phụ thuộc chặt (tightly coupled) vào implementation cụ thể.
- Dùng DI:
  - `OrderService` chỉ khai báo cần một `IOrderRepository` (interface), framework/nơi khởi tạo sẽ inject implementation cụ thể (`SqlOrderRepository`, `InMemoryOrderRepository`…) vào `OrderService`.

### Lợi ích chính

- **Giảm coupling, tăng khả năng thay thế implementation**:
  - Dễ đổi từ `SqlOrderRepository` sang `MongoOrderRepository`/`MockOrderRepository` chỉ bằng cách đổi cấu hình binding, không sửa code business.
- **Dễ test (testability)**:
  - Trong unit test, ta inject fake/mocked dependency vào, không cần DB thật, không cần gọi API thật.
- **Rõ ràng về dependencies**:
  - Nhìn vào constructor là thấy class phụ thuộc vào những gì → dễ review, dễ detect “God class”.
- **Hỗ trợ áp dụng SOLID (đặc biệt là DIP)**:
  - Class phụ thuộc vào abstraction (interface) hơn là concrete implementation.

### Các cách inject phổ biến

- **Constructor Injection** (phổ biến nhất, được khuyến nghị):
  - Dependencies được truyền qua constructor, đảm bảo object luôn ở trạng thái đầy đủ dependency ngay khi được tạo.
- **Setter Injection**:
  - Dùng setter method để gán dependencies sau khi object được tạo; linh hoạt hơn nhưng dễ dẫn tới object “nửa vời” nếu quên set.
- **Method Injection**:
  - Dependency được truyền trực tiếp qua tham số method khi cần; phù hợp cho dependency chỉ dùng trong 1–2 method cụ thể.

### DI Container / Framework

- Trong các framework như **NestJS, Spring, .NET Core**:
  - Có **IoC container** quản lý lifecycle của objects (provider/bean/service).
  - Ta đăng ký mapping `interface → implementation` (hoặc `token → provider`), container chịu trách nhiệm:
    - Khởi tạo đối tượng theo đúng scope (singleton, request-scoped, transient…).
    - Resolve cây dependency (dependency graph), phát hiện vòng lặp.
  - Ở NestJS, DI dựa trên **decorator + metadata** (`@Injectable`, `@Inject`, `@Module`), còn ở Spring là `@Component`, `@Service`, `@Repository`, `@Autowired`…

### Mối quan hệ với Clean Architecture & DDD

- **Trong Clean Architecture**:
  - DI giúp implement Dependency Rule: inner layer chỉ biết abstraction; outer layer binding implementation cụ thể và inject vào.
- **Trong DDD**:
  - Domain layer phụ thuộc vào interface (repository, service), còn infrastructure layer implement và được inject vào application services.

### Khi nào nên (và không nên) dùng DI

- **Nên dùng**:
  - Ứng dụng trung bình đến lớn, nhiều lớp service/repository, nhiều implementation cho cùng một abstraction.
  - Khi cần test unit một cách nghiêm túc, cần mock/fake dependencies.
- **Không cần quá phức tạp**:
  - Script nhỏ, app rất đơn giản có thể không cần DI container; chỉ cần viết rõ ràng, tách hàm là đủ.
