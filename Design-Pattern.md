# Bốn tính chất OOP (Object-Oriented Programming)

Bốn tính chất cốt lõi của lập trình hướng đối tượng là **Đóng gói (Encapsulation)**, **Trừu tượng (Abstraction)**, **Kế thừa (Inheritance)** và **Đa hình (Polymorphism)**. Dưới đây là cách sử dụng và ví dụ bằng Java.

---

## 1. Đóng gói (Encapsulation)

### Là gì?
- Gói dữ liệu (fields) và hành vi (methods) thao tác trên dữ liệu đó vào trong một đơn vị (class).
- **Ẩn chi tiết bên trong**: không cho truy cập trực tiếp vào dữ liệu từ bên ngoài, mà phải qua các method (getter/setter hoặc method nghiệp vụ).
- Giúp bảo vệ dữ liệu khỏi thay đổi tùy tiện và dễ bảo trì (đổi implementation bên trong mà không ảnh hưởng code gọi).

### Cách sử dụng
- Dùng **access modifier**: `private` cho field, `public` (hoặc `protected`) cho method cần expose.
- Cung cấp getter/setter khi cần đọc/ghi có kiểm soát; hoặc chỉ expose method nghiệp vụ, không expose field.

### Ví dụ Java

```java
public class BankAccount {
    private String accountId;
    private double balance;

    public BankAccount(String accountId, double initialBalance) {
        this.accountId = accountId;
        this.balance = initialBalance;
    }

    public double getBalance() {
        return balance;
    }

    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }

    public boolean withdraw(double amount) {
        if (amount > 0 && amount <= balance) {
            balance -= amount;
            return true;
        }
        return false;
    }
}
```

- `balance` là `private`, bên ngoài không gán trực tiếp `account.balance = -100`.
- Chỉ thay đổi qua `deposit`, `withdraw` (có validate) và đọc qua `getBalance()`.

---

## 2. Trừu tượng (Abstraction)

### Là gì?
- Chỉ expose **những gì cần thiết** cho người dùng, **ẩn chi tiết phức tạp** bên trong.
- Trong Java: dùng **abstract class** hoặc **interface** để định nghĩa contract (hành vi); class cụ thể implement chi tiết.

### Cách sử dụng
- Định nghĩa interface/abstract class cho nhóm đối tượng có cùng hành vi.
- Code phía gọi chỉ phụ thuộc vào interface/abstract class, không phụ thuộc class cụ thể → dễ thay thế implementation.

### Ví dụ Java

```java
// Abstraction: chỉ định nghĩa "làm gì", không định nghĩa "làm thế nào"
public interface PaymentGateway {
    boolean charge(double amount, String currency);
}

public class StripeGateway implements PaymentGateway {
    @Override
    public boolean charge(double amount, String currency) {
        // Gọi API Stripe...
        return true;
    }
}

public class MockPaymentGateway implements PaymentGateway {
    @Override
    public boolean charge(double amount, String currency) {
        return amount > 0; // Dùng cho test
    }
}
```

- Người gọi chỉ cần `PaymentGateway`; có thể truyền `StripeGateway` (production) hoặc `MockPaymentGateway` (test) mà không đổi code gọi.

---

## 3. Kế thừa (Inheritance)

### Là gì?
- Một class (subclass/child) **kế thừa** thuộc tính và phương thức của class khác (superclass/parent).
- Cho phép tái sử dụng code và xây dựng phân cấp "is-a" (ví dụ: `Dog` is-a `Animal`).

### Cách sử dụng
- Dùng khi có quan hệ **is-a** rõ ràng; tránh kế thừa chỉ để "ăn cắp code" (ưu tiên composition khi chỉ cần tái sử dụng hành vi).
- Trong Java: `extends` cho class, `implements` cho interface.

### Ví dụ Java

```java
public class Animal {
    protected String name;

    public Animal(String name) {
        this.name = name;
    }

    public void speak() {
        System.out.println(name + " makes a sound");
    }
}

public class Dog extends Animal {
    public Dog(String name) {
        super(name);
    }

    @Override
    public void speak() {
        System.out.println(name + " barks");
    }
}

public class Cat extends Animal {
    public Cat(String name) {
        super(name);
    }

    @Override
    public void speak() {
        System.out.println(name + " meows");
    }
}
```

- `Dog`, `Cat` kế thừa `name` và có thể override `speak()` để hành vi riêng.

---

## 4. Đa hình (Polymorphism)

### Là gì?
- **Cùng một kiểu tham chiếu / cùng một hành vi** (method) nhưng **đối tượng cụ thể khác nhau** thì thực thi **khác nhau**.
- Trong Java: đa hình chủ yếu qua **override** (subclass override method của superclass) và **interface**: biến kiểu interface/superclass trỏ tới nhiều implementation khác nhau.

### Cách sử dụng
- Khai báo biến kiểu superclass/interface, gán object của subclass/implementation; gọi method chung → runtime sẽ gọi đúng implementation của object thực tế.

### Ví dụ Java

```java
// Cùng kiểu Animal, nhưng mỗi đối tượng gọi speak() cho kết quả khác nhau
public class Main {
    public static void main(String[] args) {
        Animal a1 = new Dog("Rex");
        Animal a2 = new Cat("Mimi");

        a1.speak(); // Rex barks
        a2.speak(); // Mimi meows

        // Hoặc dùng mảng/collection
        Animal[] animals = { new Dog("Rex"), new Cat("Mimi") };
        for (Animal a : animals) {
            a.speak(); // Đa hình: mỗi loại gọi đúng speak() của nó
        }
    }
}
```

- Kiểu tham chiếu là `Animal`, nhưng `a1` thực tế là `Dog` nên `a1.speak()` chạy `Dog#speak()`; `a2` là `Cat` nên chạy `Cat#speak()`. Đó là đa hình.

---

## Tóm tắt

| Tính chất    | Ý chính                                      | Công cụ Java thường dùng        |
|-------------|-----------------------------------------------|----------------------------------|
| **Đóng gói**   | Ẩn dữ liệu, truy cập qua method có kiểm soát | `private` field, getter/setter   |
| **Trừu tượng** | Ẩn chi tiết, chỉ expose contract hành vi     | `interface`, `abstract class`   |
| **Kế thừa**   | Tái sử dụng & mở rộng class theo quan hệ is-a | `extends`, `implements`          |
| **Đa hình**   | Cùng kiểu tham chiếu, nhiều hành vi cụ thể   | Override, biến kiểu interface/superclass |

---

## Nạp chồng (Overloading) và Ghi đè (Overriding)

Hai cơ chế đều liên quan tới **method** nhưng khác bối cảnh: **nạp chồng** là trong **cùng một class** (hoặc phạm vi compile-time), **ghi đè** là giữa **class con và class cha** (runtime).

### Nạp chồng hàm (Overloading)

**Khái niệm:** Trong **cùng một class**, có nhiều method **cùng tên** nhưng **khác danh sách tham số** (số lượng, kiểu, thứ tự). Trình biên dịch chọn method nào gọi dựa trên đối số tại **compile-time** (static polymorphism).

**Đặc điểm:**
- Cùng tên method, khác **signature** (tham số).
- Không bắt buộc quan hệ kế thừa.
- **Return type** có thể khác nhau nhưng không dùng return type để phân biệt overload (chỉ dựa vào tham số).
- Xảy ra tại **compile-time**.

**Ví dụ Java:**

```java
public class Calculator {
    public int add(int a, int b) {
        return a + b;
    }

    public double add(double a, double b) {
        return a + b;
    }

    public int add(int a, int b, int c) {
        return a + b + c;
    }
}

// Sử dụng
Calculator cal = new Calculator();
cal.add(1, 2);       // gọi add(int, int)  → 3
cal.add(1.0, 2.0);   // gọi add(double, double) → 3.0
cal.add(1, 2, 3);    // gọi add(int, int, int)  → 6
```

---

### Ghi đè (Overriding)

**Khái niệm:** **Subclass** định nghĩa lại method đã có ở **superclass** với **cùng tên, cùng tham số (signature)**. Khi gọi method qua biến kiểu superclass trỏ tới object subclass, JVM chọn implementation của **class thực tế** tại **runtime** (dynamic polymorphism).

**Đặc điểm:**
- Cùng tên **và** cùng signature (tham số) với method ở class cha.
- Chỉ xảy ra trong quan hệ **kế thừa** (extends / implements).
- Trong Java: dùng `@Override` để rõ ràng; access modifier của method con không được hẹp hơn cha (ví dụ cha `public` thì con không thể `private`).
- Xảy ra tại **runtime** — quyết định gọi method của object thực tế.

**Ví dụ Java:**

```java
public class Animal {
    public void speak() {
        System.out.println("Animal makes a sound");
    }
}

public class Dog extends Animal {
    @Override
    public void speak() {
        System.out.println("Dog barks");
    }
}

// Sử dụng
Animal a = new Dog();
a.speak();  // Runtime: object là Dog → gọi Dog#speak() → "Dog barks"
```

- Kiểu biến là `Animal`, nhưng object thực tế là `Dog` nên `speak()` chạy phiên bản của `Dog`.

---

### Bảng so sánh nhanh

| Tiêu chí | Nạp chồng (Overloading) | Ghi đè (Overriding) |
|----------|-------------------------|----------------------|
| **Phạm vi** | Cùng class (hoặc scope) | Class con vs class cha |
| **Tên method** | Cùng tên | Cùng tên |
| **Tham số** | **Khác** (số lượng / kiểu / thứ tự) | **Giống** (cùng signature) |
| **Quan hệ** | Không cần kế thừa | Bắt buộc có kế thừa |
| **Thời điểm chọn method** | Compile-time | Runtime |
| **Mục đích** | Nhiều cách gọi cùng hành vi (tiện API) | Thay đổi hành vi của subclass so với cha |

### Tóm tắt trả lời phỏng vấn

- **Nạp chồng**: cùng class, cùng tên method, **khác tham số**; compiler chọn theo đối số lúc biên dịch.
- **Ghi đè**: class con định nghĩa lại method **cùng tên cùng tham số** với class cha; JVM chọn theo **kiểu object thực tế** lúc chạy (đa hình).

---

## Từ khóa `super` (Java) — là gì, khi nào dùng

**`super`** dùng trong **subclass** để tham chiếu tới **class cha (superclass)**: gọi constructor của cha, gọi method của cha, hoặc truy cập field của cha.

### Các cách dùng

| Cách dùng | Ý nghĩa | Ví dụ |
|-----------|---------|--------|
| **`super()`** | Gọi **constructor** của class cha (phải nằm dòng đầu trong constructor của con, nếu có). | `super(name);` trong constructor của `Dog` |
| **`super.method(...)`** | Gọi **method** của class cha (thường dùng khi con đã override method đó nhưng muốn tái sử dụng logic của cha). | `super.speak();` trong body của `Dog#speak()` |
| **`super.field`** | Truy cập **field** của class cha (khi con có field cùng tên, tránh nhầm với field của con). | `super.name` |

### Khi nào sử dụng

- **`super(...)` trong constructor:**  
  Khi class cha **có constructor có tham số**; class con bắt buộc phải gọi một constructor của cha (mặc định hoặc có tham số). Nếu cha không có constructor mặc định, con phải gọi rõ `super(...)` với đúng tham số.

- **`super.method()`:**  
  Khi subclass **override** method của cha nhưng vẫn muốn **chạy logic của cha** (mở rộng trước/sau, hoặc wrap). Ví dụ: override `toString()` nhưng vẫn in phần của cha rồi thêm phần của con.

- **`super.field`:**  
  Khi subclass khai báo field **trùng tên** với cha và cần truy cập field của cha (ít gặp; thường tránh đặt tên trùng).

### Ví dụ Java

```java
public class Animal {
    protected String name;

    public Animal(String name) {
        this.name = name;
    }

    public void speak() {
        System.out.println(name + " makes a sound");
    }
}

public class Dog extends Animal {
    private String breed;

    public Dog(String name, String breed) {
        super(name);   // Bắt buộc gọi constructor của Animal trước
        this.breed = breed;
    }

    @Override
    public void speak() {
        super.speak();  // Gọi logic của Animal trước
        System.out.println("Breed: " + breed);
    }
}
```

- **`super(name)`**: gọi `Animal(String name)` để khởi tạo `name`; phải đứng đầu constructor của `Dog`.
- **`super.speak()`**: chạy `Animal#speak()` bên trong `Dog#speak()` rồi in thêm thông tin `breed`.

### Tóm tắt trả lời phỏng vấn

- **`super`** dùng trong subclass để tham chiếu tới superclass: gọi constructor cha (`super(...)`), gọi method cha (`super.method()`), hoặc truy cập field cha (`super.field`).
- **Khi nào dùng:** (1) Constructor con bắt buộc gọi constructor cha bằng `super(...)` khi cha có constructor có tham số. (2) Trong method override, khi muốn tái sử dụng hoặc mở rộng logic của method cha bằng `super.method()`.

---

## So sánh Interface và Abstract Class (Java)

Cả hai đều dùng để **trừu tượng hóa** (định nghĩa contract, ẩn chi tiết), nhưng khác nhau về kế thừa, thành phần và mục đích sử dụng.

### Interface

| Khía cạnh | Mô tả |
|-----------|--------|
| **Định nghĩa** | Contract thuần túy: chỉ khai báo method (và từ Java 8: `default`, `static`), không có state (field instance). |
| **Kế thừa** | Một class có thể **implement nhiều interface** → hỗ trợ “đa kế thừa kiểu contract”. |
| **Thành phần** | Method abstract (implicit), từ Java 8: `default` method có body, `static` method; từ Java 9: `private` method. **Không có** constructor, **không có** field instance (chỉ `public static final` constant). |
| **Từ khóa** | `interface`, `implements`. |

**Ví dụ:**

```java
public interface Drawable {
    void draw();
    default void print() { System.out.println("Printing..."); }
}

public interface Resizable {
    void resize(double scale);
}

public class Circle implements Drawable, Resizable {
    @Override
    public void draw() { /* ... */ }
    @Override
    public void resize(double scale) { /* ... */ }
}
```

- `Circle` implement **hai** interface; chỉ định nghĩa “làm gì”, không chia sẻ code state giữa các implementation.

---

### Abstract Class

| Khía cạnh | Mô tả |
|-----------|--------|
| **Định nghĩa** | Là một **class** nhưng không khởi tạo được (`new`); có thể chứa field, constructor, method có body và method abstract. |
| **Kế thừa** | Một class chỉ có thể **extends một** abstract class (single inheritance). |
| **Thành phần** | Field (state), constructor, method abstract và method có body; có thể có `protected`/`private` member để chia sẻ cho subclass. |
| **Từ khóa** | `abstract class`, `extends`. |

**Ví dụ:**

```java
public abstract class Animal {
    protected String name;

    public Animal(String name) {
        this.name = name;
    }

    public abstract void speak();

    public void sleep() {
        System.out.println(name + " is sleeping");
    }
}

public class Dog extends Animal {
    public Dog(String name) { super(name); }
    @Override
    public void speak() { System.out.println(name + " barks"); }
}
```

- Subclass **dùng chung** state (`name`) và logic (`sleep()`); chỉ phần khác nhau mới abstract (`speak()`).

---

### Bảng so sánh nhanh

| Tiêu chí | Interface | Abstract Class |
|----------|-----------|----------------|
| Số lượng kế thừa | Nhiều (`implements A, B, C`) | Một (`extends` một class) |
| Field / state | Không (chỉ constant) | Có (instance field) |
| Constructor | Không | Có |
| Method có body | Chỉ `default`/`static` (từ Java 8) | Có, bình thường |
| Method abstract | Có (implicit) | Có (khai báo `abstract`) |
| Mục đích | Định nghĩa **contract** (capability); nhiều class không cùng gốc có thể implement. | Chia sẻ **code + state** cho nhóm class có quan hệ is-a; có logic chung. |

### Khi nào dùng gì?

- **Dùng interface** khi: chỉ cần contract (hành vi), một class cần nhiều “vai trò” (nhiều interface), hoặc các implementation không có chung base (ví dụ: `Serializable`, `Comparable`, `PaymentGateway`).
- **Dùng abstract class** khi: có **logic và state chung** cho một nhóm subclass (ví dụ: `Animal` với `name`, `sleep()`), quan hệ is-a rõ và chỉ cần một base class.

### Tóm tắt trả lời phỏng vấn

- **Interface**: contract thuần (method), không state; class có thể implement nhiều interface; dùng để định nghĩa capability/role.
- **Abstract class**: class không khởi tạo được, có thể có field + constructor + method có body + method abstract; class chỉ extends một abstract class; dùng khi có code/state chung cho cây kế thừa.

---

# Các Design Pattern: cách sử dụng và ví dụ code

Design pattern là các mẫu thiết kế tái sử dụng được để giải quyết bài toán lặp lại trong phần mềm. Dưới đây liệt kê theo nhóm **Creational**, **Structural**, **Behavioral** với cách dùng và ví dụ Java.

---

## Nhóm Creational (tạo đối tượng)

### 1. Singleton

**Là gì?** Đảm bảo một class chỉ có **một instance** duy nhất trong toàn bộ ứng dụng và cung cấp điểm truy cập toàn cục.

**Cách sử dụng:** Dùng khi cần đúng một instance: connection pool, config, logger, cache manager.

**Ví dụ Java:**

```java
public class DatabaseConnection {
    private static volatile DatabaseConnection instance;

    private DatabaseConnection() {}

    public static DatabaseConnection getInstance() {
        if (instance == null) {
            synchronized (DatabaseConnection.class) {
                if (instance == null) {
                    instance = new DatabaseConnection();
                }
            }
        }
        return instance;
    }
}
```

- Constructor `private` → bên ngoài không `new` được.
- `getInstance()` trả về cùng một instance; double-check locking cho thread-safe (hoặc dùng enum singleton đơn giản hơn).

---

### 2. Factory Method

**Là gì?** Định nghĩa **method** để tạo object, nhưng để **subclass** quyết định class cụ thể được tạo (không dùng `new` trực tiếp ở client).

**Cách sử dụng:** Khi logic tạo object phức tạp hoặc phụ thuộc điều kiện/subclass; client chỉ gọi factory method, không biết class cụ thể.

**Ví dụ Java:**

```java
public abstract class Dialog {
    public void show() {
        Button button = createButton();
        button.render();
    }
    protected abstract Button createButton();
}

public class WindowsDialog extends Dialog {
    @Override
    protected Button createButton() {
        return new WindowsButton();
    }
}

public class WebDialog extends Dialog {
    @Override
    protected Button createButton() {
        return new HtmlButton();
    }
}
```

- Client dùng `Dialog dialog = new WindowsDialog(); dialog.show();` — không cần biết `WindowsButton` hay `HtmlButton` được tạo bên trong.

---

### 3. Abstract Factory

**Là gì?** Tạo **họ sản phẩm liên quan** (nhiều loại object) mà không chỉ định class cụ thể. Mỗi factory cụ thể tạo ra một “bộ” sản phẩm tương thích.

**Cách sử dụng:** Khi có nhiều product cùng theme/họ (GUI: Button + Checkbox; UI theme Light/Dark) và cần đảm bảo dùng đúng bộ.

**Ví dụ Java:**

```java
public interface GUIFactory {
    Button createButton();
    Checkbox createCheckbox();
}

public class WinFactory implements GUIFactory {
    @Override
    public Button createButton() { return new WinButton(); }
    @Override
    public Checkbox createCheckbox() { return new WinCheckbox(); }
}

public class MacFactory implements GUIFactory {
    @Override
    public Button createButton() { return new MacButton(); }
    @Override
    public Checkbox createCheckbox() { return new MacCheckbox(); }
}

// Client
GUIFactory factory = new MacFactory();
Button btn = factory.createButton();
Checkbox cb = factory.createCheckbox();
```

- Đổi theme chỉ cần đổi `factory` (Mac vs Win), tất cả widget đi theo cùng họ.

---

### 4. Builder

**Là gì?** Tách **bước xây dựng** object phức tạp (nhiều field, nhiều optional) ra khỏi class; client gọi từng bước hoặc chuỗi method để tạo object hoàn chỉnh.

**Cách sử dụng:** Object có nhiều tham số, nhiều optional; cần code dễ đọc, tránh constructor quá dài hoặc nhiều overload.

**Ví dụ Java:**

```java
public class User {
    private final String name;
    private final String email;
    private final int age;
    private final boolean active;

    private User(Builder b) {
        this.name = b.name;
        this.email = b.email;
        this.age = b.age;
        this.active = b.active;
    }

    public static class Builder {
        private String name;
        private String email;
        private int age = 0;
        private boolean active = true;

        public Builder name(String name) { this.name = name; return this; }
        public Builder email(String email) { this.email = email; return this; }
        public Builder age(int age) { this.age = age; return this; }
        public Builder active(boolean active) { this.active = active; return this; }

        public User build() {
            return new User(this);
        }
    }
}

// Client
User u = new User.Builder()
    .name("Alice")
    .email("alice@example.com")
    .age(25)
    .build();
```

---

## Nhóm Structural (cấu trúc đối tượng)

### 5. Adapter

**Là gì?** Cho phép **interface không tương thích** của một class được dùng thông qua một lớp trung gian (adapter) chuyển đổi sang interface mà client mong đợi.

**Cách sử dụng:** Tích hợp legacy/third-party có API khác với interface hiện tại; muốn client chỉ gọi interface chuẩn.

**Ví dụ Java:**

```java
public interface MediaPlayer {
    void play(String fileName);
}

// Thư viện bên ngoài có interface khác
public class AdvancedMediaPlayer {
    public void playVlc(String file) { /* ... */ }
    public void playMp4(String file) { /* ... */ }
}

public class MediaAdapter implements MediaPlayer {
    private AdvancedMediaPlayer advancedPlayer = new AdvancedMediaPlayer();

    @Override
    public void play(String fileName) {
        if (fileName.endsWith(".mp4")) advancedPlayer.playMp4(fileName);
        else if (fileName.endsWith(".vlc")) advancedPlayer.playVlc(fileName);
    }
}
```

- Client gọi `MediaPlayer player = new MediaAdapter(); player.play("x.mp4");` — không cần biết `AdvancedMediaPlayer`.

---

### 6. Decorator

**Là gì?** “Bọc” thêm hành vi quanh một object bằng các class decorator có cùng interface; mở rộng chức năng **không cần subclass** vô tận.

**Cách sử dụng:** Thêm responsibility linh hoạt (logging, compression, encryption quanh stream/component); tránh explosion subclass.

**Ví dụ Java:**

```java
public interface Coffee {
    String getDescription();
    double cost();
}

public class SimpleCoffee implements Coffee {
    @Override
    public String getDescription() { return "Simple coffee"; }
    @Override
    public double cost() { return 1.0; }
}

public abstract class CoffeeDecorator implements Coffee {
    protected final Coffee wrapped;

    public CoffeeDecorator(Coffee w) { this.wrapped = w; }
}

public class MilkDecorator extends CoffeeDecorator {
    public MilkDecorator(Coffee w) { super(w); }
    @Override
    public String getDescription() { return wrapped.getDescription() + ", milk"; }
    @Override
    public double cost() { return wrapped.cost() + 0.5; }
}

// Client
Coffee c = new MilkDecorator(new SimpleCoffee());
System.out.println(c.getDescription() + " = " + c.cost()); // Simple coffee, milk = 1.5
```

---

### 7. Facade

**Là gì?** Cung cấp **interface đơn giản** cho một tập subsystem phức tạp; client gọi một vài method facade thay vì gọi trực tiếp nhiều class bên trong.

**Cách sử dụng:** Giảm độ phức tạp khi dùng thư viện/hệ thống con; một điểm vào thống nhất.

**Ví dụ Java:**

```java
public class OrderFacade {
    private InventoryService inventory;
    private PaymentService payment;
    private ShippingService shipping;

    public void placeOrder(String productId, int qty, String cardNumber) {
        if (!inventory.check(productId, qty)) throw new RuntimeException("Out of stock");
        payment.charge(cardNumber, inventory.getPrice(productId) * qty);
        shipping.ship(productId, qty);
    }
}
```

- Client chỉ gọi `facade.placeOrder(...)` thay vì gọi inventory, payment, shipping riêng lẻ.

---

### 8. Proxy

**Là gì?** Một object **đại diện** cho object khác, kiểm soát truy cập (lazy load, cache, kiểm tra quyền, logging).

**Cách sử dụng:** Lazy loading, caching, access control, logging trước/sau khi gọi object thật.

**Ví dụ Java:**

```java
public interface Image {
    void display();
}

public class RealImage implements Image {
    private String filename;
    public RealImage(String f) { this.filename = f; loadFromDisk(); }
    private void loadFromDisk() { /* ... */ }
    @Override
    public void display() { System.out.println("Display " + filename); }
}

public class ProxyImage implements Image {
    private RealImage real;
    private String filename;

    public ProxyImage(String f) { this.filename = f; }

    @Override
    public void display() {
        if (real == null) real = new RealImage(filename);
        real.display();
    }
}
```

- Client dùng `Image img = new ProxyImage("photo.jpg");` — chỉ khi `display()` mới load và tạo `RealImage`.

---

## Nhóm Behavioral (hành vi)

### 9. Strategy

**Là gì?** Định nghĩa **họ thuật toán**, đóng gói từng cái thành class, và cho phép **thay thế** lẫn nhau. Context dùng strategy qua interface, không biết implementation cụ thể.

**Cách sử dụng:** Nhiều cách làm cùng một việc (sort, payment, discount); cần đổi hành vi lúc runtime mà không đổi context.

**Ví dụ Java:**

```java
public interface SortStrategy {
    void sort(int[] arr);
}

public class QuickSort implements SortStrategy {
    @Override
    public void sort(int[] arr) { /* QuickSort */ }
}

public class BubbleSort implements SortStrategy {
    @Override
    public void sort(int[] arr) { /* BubbleSort */ }
}

public class Sorter {
    private SortStrategy strategy;
    public void setStrategy(SortStrategy s) { this.strategy = s; }
    public void sort(int[] arr) { strategy.sort(arr); }
}

// Client
Sorter s = new Sorter();
s.setStrategy(new QuickSort());
s.sort(data);
```

---

### 10. Observer

**Là gì?** Một object (subject) giữ danh sách **observer**; khi state thay đổi, subject **thông báo** cho tất cả observer (push hoặc pull).

**Cách sử dụng:** Event-driven UI, pub/sub, reactive: nhiều listener cần cập nhật khi dữ liệu thay đổi.

**Ví dụ Java:**

```java
public interface Observer {
    void update(String message);
}

public interface Subject {
    void attach(Observer o);
    void detach(Observer o);
    void notifyObservers();
}

public class NewsAgency implements Subject {
    private List<Observer> observers = new ArrayList<>();
    private String news;

    @Override
    public void attach(Observer o) { observers.add(o); }
    @Override
    public void detach(Observer o) { observers.remove(o); }

    public void setNews(String news) {
        this.news = news;
        notifyObservers();
    }

    @Override
    public void notifyObservers() {
        for (Observer o : observers) o.update(news);
    }
}

public class NewsChannel implements Observer {
    @Override
    public void update(String message) {
        System.out.println("Received: " + message);
    }
}
```

---

### 11. Command

**Là gì?** Đóng gói **request** thành object (command) với method `execute()`; cho phép queue, log, undo, macro.

**Cách sử dụng:** Cần queue job, undo/redo, tách người gửi request và người thực thi.

**Ví dụ Java:**

```java
public interface Command {
    void execute();
}

public class LightOnCommand implements Command {
    private Light light;
    public LightOnCommand(Light light) { this.light = light; }
    @Override
    public void execute() { light.on(); }
}

public class RemoteControl {
    private Command command;
    public void setCommand(Command c) { this.command = c; }
    public void pressButton() { command.execute(); }
}
```

---

### 12. Template Method

**Là gì?** Trong abstract class định nghĩa **skeleton** của thuật toán (template method), một số bước để **subclass override**; giữ flow chung, thay đổi chi tiết từng bước.

**Cách sử dụng:** Nhiều class có flow giống nhau, chỉ khác vài bước; tránh lặp code và đảm bảo thứ tự bước.

**Ví dụ Java:**

```java
public abstract class DataProcessor {
    public final void process() {
        readData();
        processData();
        writeData();
    }
    protected abstract void readData();
    protected abstract void processData();
    protected void writeData() {
        System.out.println("Writing result...");
    }
}

public class CsvProcessor extends DataProcessor {
    @Override
    protected void readData() { System.out.println("Reading CSV"); }
    @Override
    protected void processData() { System.out.println("Processing CSV"); }
}
```

- `process()` gọi lần lượt read → process → write; subclass chỉ implement read/process (và có thể override write).

---

## Bảng tóm tắt Design Pattern

| Pattern          | Nhóm       | Mục đích chính                          |
|-----------------|------------|------------------------------------------|
| Singleton       | Creational | Một instance duy nhất toàn app          |
| Factory Method  | Creational | Subclass quyết định class được tạo      |
| Abstract Factory| Creational | Tạo họ sản phẩm liên quan               |
| Builder         | Creational | Xây object phức tạp từng bước          |
| Adapter         | Structural | Chuyển interface không tương thích      |
| Decorator       | Structural | Bọc thêm hành vi không dùng subclass    |
| Facade          | Structural | Interface đơn giản cho subsystem        |
| Proxy           | Structural | Đại diện, kiểm soát truy cập/lazy      |
| Strategy        | Behavioral | Đổi thuật toán lúc runtime              |
| Observer        | Behavioral | Thông báo khi state thay đổi            |
| Command         | Behavioral | Đóng gói request thành object           |
| Template Method | Behavioral | Skeleton thuật toán, subclass điền bước |
