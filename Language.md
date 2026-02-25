# Cẩm nang Phỏng vấn JavaScript và NodeJS

Tài liệu này tổng hợp các câu hỏi phỏng vấn phổ biến nhất về JavaScript và NodeJS, được chia làm các khái niệm cốt lõi, từ cơ bản đến nâng cao.

---

## Phần 1: JavaScript (Kiến thức cốt lõi)

### 1. `var`, `let` và `const` khác nhau như thế nào?

- **`var`**: Có tính chất `function scope` (chỉ giới hạn phạm vi trong hàm) hoặc `global scope`. Có thể khai báo lại (re-declare) và cập nhật giá trị. Đặc biệt, `var` có tính chất **Hoisting** (được đưa lên đầu scope khi biên dịch) và khởi tạo với giá trị `undefined`.
- **`let`**: Có tính chất `block scope` (giới hạn trong cặp ngoặc nhọn `{}`). Có thể cập nhật nhưng không thể khai báo lại trong cùng một scope. Cũng bị hoisting nhưng rơi vào vùng "Temporal Dead Zone" (TDZ) - truy cập trước khi khởi tạo sẽ sinh lỗi.
- **`const`**: Tương tự `let` nhưng không thể cập nhật (re-assign) hay khai báo lại. Tuy nhiên, nếu gán bằng một `Object` hay `Array`, ta vẫn có thể thay đổi các thuộc tính/thành phần bên trong nó.

### 2. Event Loop (Vòng lặp sự kiện) trong JavaScript là gì?

**Event Loop** là một trong những khái niệm quan trọng bậc nhất để hiểu cách JavaScript và NodeJS hoạt động.

#### Vấn đề cốt lõi (Tại sao lại cần Event Loop?)

- Bản chất JavaScript là một ngôn ngữ **Single-Threaded (Đơn luồng)**, nghĩa là nó chỉ có duy nhất một Call Stack (ngăn xếp gọi hàm), và chỉ có thể thực thi **1 công việc tại 1 thời điểm**.
- Vậy nếu có 1 tác vụ tải file mất 10 giây, toàn bộ chương trình (hoặc cả giao diện trình duyệt) có bị treo cứng (blocking) trong 10 giây đó không?
- Câu trả lời là **KHÔNG**. JS giải quyết bài toán này nhờ vào cơ chế xử lý **Bất đồng bộ (Asynchronous)** được điều phối bởi **Event Loop**.

#### Event Loop hoạt động cùng các thành phần nào?

1. **Call Stack (Ngăn xếp thao tác)**: Nơi JS thực thi mã. Các hàm sẽ được đẩy (push) vào stack khi được gọi, và lấy ra (pop) khi chạy xong. Mỗi lần chỉ chạy 1 hàm ở trên cùng.
2. **Web APIs (Trình duyệt) / C++ APIs (NodeJS)**: Các tác vụ bất đồng bộ tốn thời gian như `setTimeout`, `fetch()` (gọi mạng API), đọc/ghi File, DOM events... **KHÔNG** do trực tiếp JS xử lý. Khi Call Stack gặp những hàm này, nó sẽ đẩy công việc tốn thời gian đó sang Web APIs / C++ API làm nền (background) thay cho nó. JS tiếp tục chạy đoạn code bên dưới.
3. **Task Queues (Hàng đợi tác vụ)**: Khi Web APIs / C++ làm xong việc hậu trường, nó ném các **callback (hàm xử lý kết quả)** vào các Hàng đợi (Queues). Có 2 loại Queue chính:
   - **Macrotask Queue (Hàng đợi chính)**: Chứa callbacks của `setTimeout`, `setInterval`, DOM events (click, scroll...).
   - **Microtask Queue (Hàng đợi ưu tiên)**: Chứa callbacks của `Promise` (.then/catch), `MutationObserver`.

#### Quy trình làm việc của Event Loop

**Event Loop** là một người giám sát (như một vòng lặp `while(true)`) có 2 nhiệm vụ:

1. Nhìn vào **Call Stack** xem nó đã trống chưa. Nếu đang bận thì chờ.
2. Nếu Call Stack **TRỐNG**, nó sẽ nhìn xuống các **Queues (Hàng đợi)** để lấy công việc tiếp theo đưa lên Call Stack chạy.

**Thứ tự ưu tiên:**

- **Bước 1**: Chạy các code đồng bộ (synchronous) đang có trên **Call Stack** cho đến khi không còn gì.
- **Bước 2**: Khi Call Stack trống, Event Loop ưu tiên nhìn vào **Microtask Queue**. Nó sẽ bốc **TOÀN BỘ** các tác vụ trong Microtask Queue đẩy dần lên Call Stack chạy cho bằng hết.
- **Bước 3**: Khi Microtask Queue đã sạch, nó mới nhìn qua **Macrotask Queue**. Nó bốc đúng **MỘT** tác vụ đầu tiên đẩy lên Call Stack để chạy.
- Sau khi chạy xong 1 macrotask, Event Loop **LẠI** quay về kiểm tra xem bên Microtask Queue có gì mới xuất hiện không để ưu tiên dọn sạch lại từ đầu.

> _"Tóm lại: Event Loop là cơ chế giúp JavaScript, vốn đơn luồng, có thể xử lý các tác vụ bất đồng bộ (non-blocking) bằng cách đẩy các tác vụ nặng cho hệ thống nền (Web API / libuv) xử lý, và sử dụng một vòng lặp để liên tục điều phối các callback từ Hàng đợi (Queues) trở lại Ngăn xếp (Call Stack) thực thi khi luồng chính rảnh rỗi."_

### 3. Closure (Bao đóng) là gì?

Closure là một hiện tượng mà một hàm con **khóa (ghi nhớ)** và có thể truy cập các biến của hàm cha (lexical scope bên ngoài của nó), ngay cả sau khi hàm cha đã thực thi và kết thúc (return).

**Ứng dụng thực tế:**

- Tạo dữ liệu private (Encapsulation).
- Currying function.

```javascript
function createCounter() {
  let count = 0; // Biến này không tự mất đi do được closure giữ lại
  return function () {
    count++;
    return count;
  };
}
const counter = createCounter();
console.log(counter()); // 1
console.log(counter()); // 2
```

### 4. Phân biệt `==` (Loose equality) và `===` (Strict equality)

- **`==`**: So sánh giá trị sinh ra cơ chế ép kiểu (Type Coercion). Nếu hai vế khác kiểu, JS sẽ cố ép chúng về cùng một kiểu trước khi so sánh (Ví dụ: `1 == "1"` là `true`).
- **`===`**: So sánh một cách khắt khe cả về mặt giá trị lẫn kiểu dữ liệu. Không xảy ra ép kiểu ngầm. Luôn khuyến khích dùng `===`.

### 5. `this` trong JavaScript hoạt động ra sao?

Giá trị của `this` do cách hàm được gọi quyết định (runtime binding):

1. **Gọi hàm thông thường (Regular function call):** Trỏ vào Global object (`window` ở trình duyệt, `global` ở NodeJS) hoặc `undefined` nếu ở `strict mode`.
2. **Gọi qua Method của Object:** Trỏ vào Object gọi method đó.
3. **Arrow function:** Không có `this` riêng, nó kế thừa `this` từ phạm vi lexcial (bao ngoài) nó lúc định nghĩa hàm.
4. **Hàm dùng `call()`, `apply()`, `bind()`:** Trỏ tới object được truyền vào một cách chủ động.

### 6. Call, Apply và Bind khác nhau ở đâu?

Dùng để thay đổi ngữ cảnh (`this`) cho hàm:

- **`call(thisArg, arg1, arg2...)`**: Gọi hàm ngay lập tức, truyền tham số dưới dạng danh sách ngăn cách bởi dấu phẩy.
- **`apply(thisArg, [arg1, arg2...])`**: Gọi hàm ngay lập tức, tham số truyền vào hàm dưới dạng một **mảng**.
- **`bind(thisArg, ...)`**: Trả ra một **HÀM MỚI** với `this` đã được trói buộc vĩnh viễn, **không thực thi ngay**. Mọi người thường gán vào một biến rồi gọi sau.

### 7. Nguyên mẫu (Prototype) trong JS là gì?

Mọi object trong JS sinh ra đều có một thuộc tính ngầm liên kết nó với một object khác, gọi là Prototype. Khi chúng ta cố truy cập một thuộc tính của một object mà nó không có sẵn, JS sẽ dò tìm lên phía trên Prototype của object đó (và cứ thế tiếp tục tạo thành **Prototype Chain**) cho đến khi tìm thấy, hoặc chạm mốc kết thúc ở `null`. Cơ chế này mang lại kế thừa (Inheritance) trong JS.

### 8. Phân biệt Callback, Promise và Async/Await

Đây là 3 cách tiếp cận lịch sử trong JavaScript để xử lý các tác vụ bất đồng bộ (Asynchronous):

- **Callback**: Là một hàm được truyền vào một hàm khác dưới dạng tham số để nó được gọi (thực thi) sau khi thủ tục bất đồng bộ của hàm chính hoàn thành.
  - _Nhược điểm_: Khi có nhiều tác vụ phụ thuộc nhau, các callback bị lồng ghép quá sâu tạo thành **Callback Hell** (hay kim tự tháp hủy diệt). Code cực kỳ khó đọc, khó bảo trì và phân tán logic xử lý lỗi.
- **Promise**: Ra mắt ở ES6 để giải các vấn đề của callback. Nó là một đối tượng đại diện cho sự hoàn thành (hoặc thất bại) của một thao tác bất đồng bộ. Nó có 3 trạng thái: `Pending` (đang chờ), `Fulfilled` (thành công) hoặc `Rejected` (thất bại).
  - _Ưu điểm_: Giải quyết mã lồng nhau bằng cách nối chuỗi (chaining) các hàm `.then()`, và gom việc xử lý lỗi về một hàm `.catch()` duy nhất ở cuối.
- **Async/Await**: Sinh ra ở bản cập nhật ES8, thực chất là một lớp áo cú pháp (Syntactic sugar) bọc ngoài Promise.
  - _Ưu điểm_: Cho phép viết các đoạn code bất đồng bộ mà trông có vẻ đồng bộ, tuyến tính từ trên xuống. Không còn chuỗi `.then()`. Xử lý lỗi rất trực quan và quen thuộc với phong cách lập trình cấu trúc thông qua khối lệnh `try...catch`.

---

## Phần 2: NodeJS (Kiểu trúc và Vận hành)

### 1. NodeJS là gì và nó dựa trên kiến trúc nào?

NodeJS không phải là một ngôn ngữ lập trình hay một framework. Nó là một **môi trường thực thi (Runtime Environment)** mã nguồn mở, đa nền tảng, thiết kế để chạy các mã JavaScript (JS) ở phía máy chủ (Server-side) thay vì chỉ chạy trên trình duyệt (Client-side).

Kiến trúc của NodeJS kết hợp 2 thành phần cốt lõi:

- **V8 Engine** (Của Google Chrome): Giúp biên dịch trực tiếp mã JavaScript sang mã máy với tốc độ cực nhanh.
- **libuv**: Một thư viện C chuyên dụng cung cấp hệ thống kết nối Event Loop và cơ chế **Non-blocking I/O** (I/O bất đồng bộ) giúp Node có thể thực thi nhiều tác vụ cùng một lúc.

### 2. Điểm mạnh và Điểm yếu của NodeJS?

**Điểm mạnh (Ưu điểm):**

- **Kiến trúc Non-blocking I/O và Event-driven**: Giúp xử lý đồng thời (Concurrency) hàng ngàn, thậm chí hàng chục ngàn kết nối (request) cùng lúc mà tốn rất ít RAM do nó không phải tạo ra một CPU Thread (luồng) mới cho mỗi request như một số ngôn ngữ (Java, PHP, Ruby...).
- **Đồng nhất ngôn ngữ (Full-stack JS)**: Chỉ cần biết JavaScript, bạn có thể code cả Frontend (React/Vue) lẫn Backend và giao tiếp với Database (MongoDB). Việc này giúp team làm việc mượt mà, dễ hiểu mã nguồn của nhau và tiết kiệm chi phí học tập.
- **Tốc độ thực thi rất nhanh**: Bằng việc tận dụng V8 Engine, tốc độ xử lý các đoạn mã JS của Node là rất đáng nể.
- **Xử lý dữ liệu định dạng JSON dễ dàng**: Node tương tác cực mượt với JSON (định dạng dữ liệu phổ biến nhất hiện nay trên Internet) do JSON bắt nguồn từ JS. Việc truyền và parse dữ liệu diễn ra hoàn toàn tự nhiên không cần thư viện bên thứ 3 định hình lại.
- **Mạng lưới thư viện NPM khổng lồ**: Node Package Manager (NPM) là kho tàng chứa hàng triệu thư viện mã nguồn mở có sẵn. Bạn chỉ cần vài dòng lệnh để cài các modul xây dựng API, xử lý hình ảnh, mã hóa mật khẩu, v.v… giúp tăng tốc độ làm dự án rất nhiều.
- **Tuyệt vời cho hệ thống Real-time (thời gian thực)**: NodeJS là vua trong việc xây dựng các ứng dụng WebSockets hai chiều như Chat app, Game online, Streaming và các kiến trúc hệ thống chia nhỏ (Microservices).

**Điểm yếu (Nhược điểm):**

- **Kém xử lý các tác vụ nặng (CPU-Intensive Tasks)**: Do bản chất gốc là Single Threaded (Đơn luồng), nếu bạn bắt Node phải tính toán một thuật toán phức tạp, convert video, render ảnh 3D, hay xử lý big data bằng chính luồng đó, nó sẽ **chặn (block) hoàn toàn Event Loop**. Cả server sẽ bị "treo" chờ cho việc tính toán đó hoàn thành, làm các request của user khác đến server bị bỏ lơ.
- **Callback Hell và Code phức tạp**: Quá phụ thuộc vào cơ chế bất đồng bộ (Asynchronous) thông qua Callbacks có thể biến mã nguồn thành một đống logic lồng ghép vào nhau nếu không phân chia code cấu trúc khéo léo. Tuy nhiên nhược điểm này nay được giảm thiểu đáng kể bằng cách dùng chuẩn mới `Promise` và cú pháp `async / await`.
- **Thư viện NPM lộn xộn/kém chất lượng**: Kho NPM phong phú là ưu thế nhưng đôi khi là điểm trừ. Ai cũng có thể đẩy thư viện lên đây, dẫn đến tình trạng các thư viện lõi (dependencies) trong dự án có thể bị phá vỡ nếu bị lỗi cập nhật ngầm, hoặc đôi khi có mã độc gây ra lỗ hổng bảo mật.
- **Thiếu sự chặt chẽ về mặt kiểu (Types)**: Javascript vốn là ngôn ngữ _Loose-typed_ nên khi dự án scale lớn, khó kiểm soát lỗi sai về dữ liệu. (Đó là lý do các công ty thường kết hợp sử dụng TypeScript với NodeJS cho các dự án lớn).

### 3. NodeJS có Single Thread thực sự không? Làm sao để xử lý nhiều requests một lúc?

**NodeJS là Single Thread ở tầng lập trình viên (JS thread), nhưng có một mô hình Asynchronous Non-blocking I/O ẩn đằng sau.**

- Khi có một request mới (như chọc DB, lấy file mạng, gọi API khác), Node gửi yêu cầu trỏ xuống tầng OS hoặc `libuv` giải quyết thay vì đứng chờ (blocking).
- Sau khi bàn giao tác vụ, Node (Event Loop) lại lập tức rảnh rang để nhận và phục vụ các request mới tiếp theo ngay luồng đó.
- Khi OS hoặc `libuv` (thông qua C++ Thread Pool) xử lý xong tác vụ nặng, nó trả callback lại qua Event Queue, sau đó Event Loop sẽ pick lên xử lý tiếp lúc nào rảnh. Nhờ vậy có thể xử lý concurrency cực cao dẫu chỉ dùng 1 main JS thread.

### 4. Streams trong Node là gì? Tại sao phải dùng Stream thay cho I/O thông thường?

**Streams (Luồng)** là một trong những concept mạnh mẽ và quan trọng nhất của NodeJS khi nhắc tới xử lý data lớn.
Stream giúp lấy và thao tác (đọc/ghi) một lượng dữ liệu chia thành từng đoạn nhỏ (gọi là **chunks**) theo một "đường ống" một cách liên tục thay vì tải toàn bộ file đó vào RAM một lúc như cách I/O truyền thống (`fs.readFile`).

**Tại sao phải dùng Stream?**

1. **Tiết kiệm Bộ nhớ (Memory Efficiency)**: Tưởng tượng bạn có file video 2GB nhưng VPS server của bạn chỉ có 512MB RAM. Nếu dùng `fs.readFile` (đọc toàn bộ file vào RAM rồi mới xử lý/gửi đi), server sẽ Crash tắp lự (Out of Memory). Với stream, nó chỉ tải từng chunk nhỏ (vd 64KB) xử lý xong nhả ra tải chunk tiếp theo. Rất an toàn.
2. **Tiết kiệm Thời gian (Time Efficiency)**: Với I/O truyền thống, bạn phải đợi TẤT CẢ dữ liệu tải xong thì mới bắt đầu xử lý mảnh đầu tiên. Với stream, dữ liệu chunk đầu tiên về tới là ta có thể bắt đầu xử lý và gửi cho user ngay lập tức (giống cách Netflix hay Youtube stream video cho bạn xem mà không cần đợi tải xong cả bộ phim).

**Bốn loại Stream chính trong NodeJS:**

1. **Readable Stream:** Chỉ để đọc dữ liệu vào (Ví dụ: `fs.createReadStream()`, nhận HTTP Request ở Server).
2. **Writable Stream:** Để viết/đẩy dữ liệu ra (Ví dụ: `fs.createWriteStream()`, trả HTTP Response về Client).
3. **Duplex Stream:** Có thể vừa đọc vừa viết liền mạch, giống như một đường hầm hai chiều (Ví dụ: Socket liên kết mạng `net.Socket`).
4. **Transform Stream:** Là Duplex stream nhỉnh hơn, nó có thể lấy dữ liệu đọc vào, **CHỈNH SỬA / BIẾN ĐỔI** dữ liệu đó, rồi ghi trực tiếp ra đầu ra (Ví dụ: module `zlib` giúp vừa đọc file vừa nén nó thành file `.zip`, hoặc module mã hóa `crypto`).

_(Tip lúc phỏng vấn)_: Hai tính năng thường được kết hợp cùng Stream là **`pipe()`**, giúp nối thẳng đầu ra của Readable Stream vào đầu vào của một Writable Stream một cách nhàn hạ mà khỏi cần quản lý dữ liệu lắt nhắt.

### 5. Khác biệt giữa `process.nextTick()` và `setImmediate()`

- **`process.nextTick(callback)`**: Dù không nằm trong các Phase chính thức của Event Loop, callback ở nextTick có **prioriry lớn nhất**, chạy MỘT CÁCH TỨC THỜI ngay khi phase hiện tại của Event Loop dừng lại (trước chạy cả Microstasks array như Promise). Dùng khi muốn một logic phải chạy liền trước khi I/O tiếp theo đụng đến.
- **`setImmediate(callback)`**: Được chạy ở một Phase đặc thù có tên là **Check Phase** của vòng lặp Event Loop, chạy sau kết thúc của I/O callbacks và Polling Phase.

### 6. ES Modules (`import/export`) khác gì so với CommonJS (`require/module.exports`)?

- **CommonJS (`require`)**: Khởi thủy mặc định của NodeJS. Nó diễn ra **đồng bộ (synchronous)** tại thời điểm hoạt động (runtime). Nếu require 1 file quá bự nó sẽ dừng tải các bước dưới.
- **ES Modules (`import`)**: Standard hiện đại, syntax tương đồng Frontend (React/Vue). Nó diễn ra **bất đồng bộ**, chạy ở bước phân tích mã (static analysis/parse-time), hỗ trợ **Tree-shaking** (tối ưu xoá bỏ function không xài lúc build) xuất sắc hơn.

### 7. Các cách xử lý Lỗi (Error Handling) trong NodeJS

1. Dùng quy ước **Error-First Callback** (áp dụng với API Node cổ điển): param đầu của callback là lỗi, các data là param đính sau. `(err, data) => {...}`
2. Triển khai **`try { } catch (error) { }`** cho xử lý đồng bộ và hàm `async / await`.
3. Dùng `.catch()` cho các **Promises**.
4. Sử dụng events phòng hờ crash app ngầm do forgot catch:
   - `process.on('uncaughtException', ...)`
   - `process.on('unhandledRejection', ...)`
     _(Chỉ dùng bắt lỗi global này để ghi Log (winston/pino), xoá session rồi restart process một cách an toàn (graceful shutdown), tuyệt đối không để chôn luôn cái lỗi và tiếp tục chạy vì state chương trình đã corrupt)._

### 8. So sánh NodeJS và Java (Spring Boot)

Đây là câu hỏi thường xuyên gặp khi ứng tuyển Backend, yêu cầu ứng viên hiểu rào cản và sức mạnh của 2 ecosystem phổ biến nhất:

| Tính năng cốt lõi                    | NodeJS                                                                                                                                                            | Java (Spring Boot)                                                                                                                                                                 |
| ------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Mô hình luồng (Threading)**        | **Single-Threaded** (Đơn luồng) kết hợp Event Loop (Non-blocking I/O). Rất mượt để handle lượng request khổng lồ nhưng sợ CPU calculation.                        | **Multi-Threaded** (Đa luồng). Mỗi request mới thường sinh ra 1 thread riêng biệt. Cực mạnh khi tính toán, nhưng tốn tải RAM/Memory lớn cho các thread.                            |
| **Kiểu dữ liệu (Typing)**            | Tự do (Dynamically typed). Rất nhanh để xây dựng Prototype nhưng rình rập nhiều lỗi vặt Runtime nếu project phình to (Có thể giải quyết phần nào bởi TypeScript). | Rất chặt chẽ (Statically typed). Rất mất thời gian khai báo ban đầu nhưng bù lại an toàn 100% thời điểm compile, siêu phù hợp cho hệ thống doanh nghiệp lõi (Ngân hàng, bảo hiểm). |
| **Hiệu suất / Tốc độ**               | Rất nhanh ở các truy vấn I/O (Database access, Network call), phù hợp làm API Server, Microservices, App Chat thời gian thực.                                     | Nhỉnh hơn tuyệt đối khi bị ép phải thực thi các tác vụ tính toán (Toán học phức tạp, Xử lý video, Big Data).                                                                       |
| **Hệ sinh thái (Ecosystem)**         | Kho NPM khổng lồ với hàng triệu thư viện, nhiều tool mới mẻ hiện đại. Nhược điểm là rác cũng nhiều, dependencies chằng chịt dễ lỗi vặt.                           | Khủng khiếp và vững chãi. Maven/Gradle sở hữu các thư viện cực kỳ bảo mật (Security), ổn định suốt hàng thập kỷ được các tập đoàn lớn tin dùng.                                    |
| **Độ khó tiếp cận (Learning Curve)** | Rất thấp. Cú pháp JS quá quen thuộc, Fullstack Dev dễ nhảy qua nhảy lại giữa Client và Server.                                                                    | Độ khó cao. Cần hiểu sâu về OOP (Lập trình hướng đối tượng), Design Patterns, cấu trúc dự án phức tạp hơn nhiều.                                                                   |

**Tóm tắt khi tư vấn dự án:**

- Hãy tư vấn **NodeJS** khi: Startup làm App cần ra lò khẩn cấp (MVP), Team chuyên React/JS, Dùng MongoDB/NoSQL nhiều, App mang tính Streaming, Chat, Single Page Application.
- Hãy tư vấn **Java** khi: Làm cho dự án chính phủ / Enterprise (Ngân hàng), Kiến trúc cực kỳ phức tạp (Architecture nặng), DB phức mang tính Relational (SQL) đồ sộ, Chú trọng tính bảo mật vững chãi và vận hành thập kỷ.

### 9. Worker Threads trong NodeJS là gì và hoạt động ra sao?

**Khái niệm:**  
Như đã đề cập, NodeJS là **Single-Threaded** (đơn luồng), có nghĩa là nó gặp giới hạn khổng lồ và sẽ bị "treo" (blocking Event Loop) nếu phải tính toán các tác vụ nặng (CPU-intensive tasks) như xử lý ảnh 3D, mã hóa/giải mã video, hay tính toán AI... Để giải quyết sự yếu kém này, NodeJS đã chính thức giới thiệu **Worker Threads** (thông qua module `worker_threads`).

**Cơ chế hoạt động:**

- Thay vì sinh ra các tiến trình (Process) hoàn toàn mới và rườm rà như module `cluster`, Worker Threads tạo ra các **luồng (Threads) phụ** nhẹ nhàng chạy song song, chia sẻ cùng nhau **một tiến trình (Process) cha** duy nhất.
- Tính năng vũ khí lợi hại nhất của Worker Threads là khả năng **Chia sẻ Bộ nhớ (Shared Memory)** qua `SharedArrayBuffer`, giúp các luồng có thể trực tiếp làm việc trên cùng một mảng dữ liệu cực lớn mà không cần thao tác tuần tự dữ liệu cồng kềnh qua lại bằng cơ chế `postMessage` (MessageChannel) tốn tài nguyên.
- Tuy dùng chung Process, nhưng bản thân mỗi Worker vẫn sở hữu một **V8 Engine độc lập, Node Environment Context cá nhân cũng như Event Loop riêng biệt**. JS code chạy trên luồng Worker không bao giờ block luồng Main.

**Tựu trung (Ghi nhớ cho phỏng vấn):**

> _Nên dùng Worker Threads để giải quyết các mớ code cần cày ải CPU (CPU-bound) để Event Loop chính luôn thông thoáng. TUYỆT ĐỐI KHÔNG NÊN lạm dụng nó để xử lý các tác vụ I/O (call API / read DB / read File) vì cốt lõi của NodeJS với cơ chế Non-blocking I/O ngầm vốn đã có tốc độ vô đối trong mảng xử lý I/O concurrent rồi._

### 10. Buffer là gì? Khác biệt giữa Buffer và Stream như thế nào?

**Buffer là gì?**

- Trong NodeJS, `Buffer` là một class (mặc định global) dùng để lưu trữ và làm việc trực tiếp với **dữ liệu nhị phân (binary data)** nguyên thủy.
- Bạn có thể hình dung Buffer giống như một **mảng các số nguyên** (mỗi số đại diện cho 1 byte dữ liệu từ 0 đến 255), nhưng kích thước của nó bị cố định (fixed-size) ngay từ lúc khởi tạo và không thể co giãn.
- Điểm đặc biệt là vùng nhớ của Buffer được cấp phát ở bên ngoài bộ nhớ Heap của V8 Engine (do tầng C++ của Node quản lý trực tiếp), giúp tốc độ cấp phát rất nhanh. Buffer tự động xuất hiện dưới nền mỗi khi ta đọc file, tải ảnh/video, hay giao tiếp mạng.

**Khác biệt giữa Buffer và Stream (Cách trả lời dễ ghi điểm bằng ẩn dụ):**

Hãy hình dung quá trình di chuyển ở bến xe buýt:

- **Buffer:** Chính là chiếc xe buýt (hay khu vực chờ). Nó gom (thu thập) hành khách (dữ liệu nhị phân) lại vào một không gian tạm thời. Chỉ khi xe đầy hành khách (hoặc đến giờ), toàn bộ chiếc xe đó mới di chuyển 1 lượt.
- **Stream:** Là thao tác dòng người đi bộ qua một ống lồng hẹp (như ra máy bay vậy). Hành khách (dữ liệu) chui qua liên tục, người này nối tiếp người kia tới đích mà không cần phải gom lại chờ bất cứ ai. (Thực chất, từng hành khách chạy qua ống lồng chính là từng **cục Buffer nhỏ lẻ (chunks)**).

**Tóm tắt sự phân biệt khi phỏng vấn:**

| Tiêu chí                  | Buffer (fs.readFile)                                                                                                                          | Stream (fs.createReadStream)                                                                                                                       |
| ------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Cách truyền thải**      | Gom tất cả dữ liệu thành 1 khối duy nhất rồi mới thao tác/gửi đi.                                                                             | Dữ liệu được cắt nhỏ (chia thành các chunks) và chảy liên tục.                                                                                     |
| **Tiêu tốn Bộ nhớ (RAM)** | **Rất nguy hiểm**. Kích thước file bao nhiêu thì ngốn bấy nhiêu RAM. Xử lý file 2GB trên VPS có 1GB bộ nhớ sẽ làm crash (Out of Memory) ngay. | **Rất tiết kiệm**. Chỉ tốn vài chục KB bộ nhớ (chính là size của mỗi chunk Buffer đang luân chuyển) là có thể thao tác với file dung lượng vô hạn. |
| **Độ trễ (Latency)**      | Phải đợi gom đủ 100% dữ liệu mới được thao tác tiếp -> Rất chậm.                                                                              | Bất cứ mảnh (chunk) dữ liệu nào chui qua ống stream là có thể thao tác và truyền đi ngay tức khắc -> Khách hàng thấy kết quả ngay lập tức.         |

### 11. Khi nào nên dùng Worker Threads? Khi nào nên dùng Child Process?

Mặc dù cả `worker_threads` và `child_process` đều là các giải pháp cốt lõi để "vượt rào" hạn chế đơn luồng (Single-Threaded) của NodeJS, nhưng mục đích áp dụng của chúng lại rất khác biệt:

1. **Child Process (Tiến trình con)**:
   - **Bản chất**: Sinh ra (fork/spawn) một quá trình hệ điều hành hoàn toàn ĐỘC LẬP. Nó có không gian bộ nhớ riêng biệt hoàn toàn.
   - **Giao tiếp**: Phải giao tiếp qua hệ thống nhắn tin (IPC - Inter Process Communication) bằng cách serialize/deserialize dữ liệu (thường sang JSON). Quá trình này rất đắt đỏ và chậm.
   - **Khi nào dùng?**
     - Dùng để chạy một file thực thi hoặc 1 ngôn ngữ độc lập khác (Ví dụ gọi 1 script Python học máy, thực thi file bash, hay chạy FFmpeg để convert video).
     - Muốn chạy các tác vụ có tính cô lập cao, nếu app đó bị crash cũng không gây hề hấn gì đến NodeJS app chính.
   - **Nhược điểm**: Tổn hao cực lớn RAM hệ thống.

2. **Worker Threads (Luồng phụ)**:
   - **Bản chất**: Tạo các luồng phụ nằm GỌN BÊN TRONG tiến trình NodeJS cha. Không sinh ra tiến trình OS mới.
   - **Giao tiếp**: Cực kỳ tối ưu nhờ khả năng chia sẻ mảng bộ nhớ dùng chung (`SharedArrayBuffer`). Không cần phải serialize phức tạp tốn kém thời gian như IPC.
   - **Khi nào dùng?**
     - Dành cho những tính toán nặng thuần túy (CPU Intensive Tasks) bằng CHÍNH mã gốc JavaScript / NodeJS (như parsing file JSON vài GB, thao tác ma trận Toán học, hoặc mã hóa hash SHA-256 cường độ cao).
   - **Ưu điểm**: Nhẹ bộ nhớ hơn hẳn, thao tác giao tiếp data giữa luồng chính và luồng phụ là siêu tốc.

**Bí kíp Tóm lược cho nhà tuyển dụng:**

> "Dùng **Child Process** nếu ta cần chạy mã của một phần mềm/ngôn ngữ khác bên ngoài, chấp nhận cách ly dung lượng RAM riêng biệt. Dùng **Worker Threads** để viết hàm tính toán Toán Học/CPU cường độ cao ngay bằng JS mà vẫn muốn chia sẻ chung một vùng biến dữ liệu dưới nền với luồng cha để tiết kiệm tài nguyên."
