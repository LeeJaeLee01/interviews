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

### 4. Streams trong Node là gì? Có mấy loại?

Streams dùng để lấy và thao tác (đọc/ghi) một lượng dữ liệu chia thành từng đoạn nhỏ (chunks) theo đường ống một cách liên tục thay vì tải toàn bộ nó vào RAM cực kỳ phí bộ nhớ.
Ví dụ: Thao tác file cực lớn, stream video.
Bốn loại Stream chính:

1. **Readable:** Chỉ đọc dữ liệu (`fs.createReadStream()`).
2. **Writable:** Để viết dữ liệu ra (`fs.createWriteStream()`).
3. **Duplex:** Có thể vừa đọc vừa viết (`net.Socket`).
4. **Transform:** Có thể vừa đọc vừa viết nhưng chỉnh sửa được chunk dữ liệu qua stream đó (ví dụ zlib giúp mã hóa/nén gzip).

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
