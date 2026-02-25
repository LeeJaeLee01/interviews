# Cẩm nang Phỏng vấn cơ sở dữ liệu (Database)

Tài liệu này tổng hợp các câu hỏi phỏng vấn thường gặp về Database (CSDL Relational / NoSQL, tối ưu hóa truy vấn).

---

### 1. Indexing (Đánh chỉ mục) là gì? Có các kiểu đánh Index (chỉ mục) nào phổ biến?

**Khái niệm:**
Index (chỉ mục) trong CSDL giống như phần Mục lục ở đầu một quyển sách. Thay vì phải lật từng trang (Full Table Scan) để tìm một thông tin, CSDL sẽ tra cứu trong cấu trúc Index nhỏ gọn trước để tìm ra chính xác "số trang" (con trỏ địa chỉ vật lý) ghi dữ liệu đó. Điều này giúp tăng tốc độ của lệnh `SELECT` (đọc) lên hàng trăm đến hàng ngàn lần.
Nhưng bù lại, nó làm chậm đi quá trình `INSERT`, `UPDATE`, `DELETE` vì mỗi khi dữ liệu thay đổi, mục lục Index cũng phải được cập nhật lại theo.

**Các kiểu đánh Index phổ biến nhất:**

#### 1. B-Tree Index (Hoặc B+Tree)

- **Đặc điểm:** Đây là kiểu Index mặc định và thiết yếu nhất trong hầu hết mọi RDBMS (MySQL, PostgreSQL...). Nó tổ chức dữ liệu dưới dạng cấu trúc cây cân bằng (Balanced Tree).
- **Khi nào dùng:** Rất mạnh để tìm kiếm các giá trị **chính xác**, so sánh (`=`, `<`, `>`), hoặc tìm kiếm theo **khoảng (range)** (ví dụ: `WHERE age BETWEEN 18 AND 30`). Tiền tố chuỗi cũng hoạt động tốt (ví dụ: `WHERE name LIKE 'Nguyen%'`).

#### 2. Hash Index

- **Đặc điểm:** Dùng thuật toán Băm (Hashing) tự động băm giá trị của cột thành ID băm trỏ thẳng đến bộ nhớ vật lý.
- **Khi nào dùng:** Cực kỳ bá đạo (nhanh hơn cả B-Tree) với tác vụ truy vấn **chính xác tuyệt đối** (như `WHERE id = 123` hoặc `WHERE email = 'abc@test.com'`).
- **Nhược điểm nặng:** Hoàn toàn bị phế võ công/không thể dùng cho các truy vấn so sánh hoặc tìm kiếm theo khoảng (`<`, `>`, `BETWEEN`, `LIKE`).

#### 3. Bitmap Index

- **Đặc điểm:** Thay vì lưu giá trị, nó dùng chuỗi bit `1` và `0` để biểu diễn sự tồn tại của 1 giá trị. (1 = Đúng, 0 = Sai).
- **Khi nào dùng:** Cực kỳ hoàn hảo, siêu tối ưu không gian bộ nhớ khi đánh Index cho các cột có dữ liệu **ít tính chất phân loại (Low Cardinality)**. Ví dụ: Cột `GioiTinh` (chỉ có Nam/Nữ), cột `TrangThai` (Thành công/Thất bại/Chờ).

#### 4. Full-Text Index

- **Đặc điểm:** Tạo ra một danh sách "Inverted Index" (Chỉ mục ngược) ánh xạ các từ khoá (words) thay vì ánh xạ các dòng dữ liệu.
- **Khi nào dùng:** Khi muốn tạo bộ Search **tìm kiếm các cụm từ nằm chen giữa một văn bản rất dài** (Ví dụ hộp công cụ tìm kiếm bài viết Blog, tìm kiếm tên hàng hoá). Nó sinh ra để thay thế cho `LIKE '%keyword%'` (vốn cực kì chậm vì phải full-scan table).

#### 5. Spatial Index (Chỉ mục không gian) / R-Tree

- **Đặc điểm:** Đánh index dựa trên toạ độ hình học (Gird/Polygons).
- **Khi nào dùng:** Xử lý các bài toán tìm kiếm định vị bản đồ như: "Tìm 5 quán cà phê gần vị trí (Kinh độ/Vĩ độ) của tôi nhất bán kính 2km". (Ứng dụng cho Grab, Tinder, Food Delivery).

#### 6. Composite / Compound Index (Chỉ mục phức hợp)

- **Đặc điểm:** Đánh index **kết hợp từ 2 cột trở lên**. (Ví dụ: `CREATE INDEX idx_name_age ON users(last_name, age)`).
- **Khi nào dùng:** Khi câu truy vấn thường xuyên phải tìm kiếm thỏa mãn nhiều điều kiện cùng lúc (Ví dụ: `WHERE last_name = 'Lee' AND age = 22`).
- **Lưu ý chí mạng:** Phải tuân theo quy tắc **Leftmost Prefix Rule** -> Tức là nếu index được tạo theo thứ tự (A, B, C) thì một câu query chỉ ăn được Index nếu nó tìm kiếm kiểu `WHERE A` hoặc `WHERE A VÀ B`. Nếu query nhảy cóc kiểu `WHERE B VÀ C` thì cái Index (A,B,C) đó hoàn toàn VÔ DỤNG.

### 2. Phân loại Index theo cấu trúc lưu trữ vật lý (Clustered vs Non-Clustered Index)

Ngoài việc phân loại theo thuật toán (B-Tree, Hash...), người phỏng vấn sẽ rất gắt gao ở câu hỏi phân biệt cấu trúc vật lý của Index.

#### 1. Clustered Index (Chỉ mục cụm)

- **Đặc điểm:** Clustered Index quyết định **thứ tự lưu trữ vật lý** của chính các dòng dữ liệu trong ổ cứng. Vì dữ liệu thật chỉ có thể được sắp xếp theo _một thứ tự duy nhất_ (như danh bạ điện thoại xếp từ A-Z), nên **mỗi bảng CHỈ CÓ THỂ CÓ TỐI ĐA 1 Clustered Index**.
- **Mối quan hệ với Primary Key:** Thông thường, khi bạn tạo `Primary Key` (Khoá chính) cho một bảng, hệ quản trị CSDL sẽ tự động tạo luôn một Clustered Index trên cột đó. (Đó là lý do tại sao tìm kiếm theo ID lại luôn cực kỳ nhanh).
- **Cấu trúc:** Node lá (Leaf node) của cây B-Tree trong Clustered Index chứa **TOÀN BỘ dữ liệu thực tế** của dòng đó (Row data).
- **Tốc độ:** Tốc độ đọc (SELECT) **siêu nhanh** vì một khi tìm thấy Index là lấy được luôn cục data ngay tại node lá mà không cần "nhảy" đi đâu tìm tiếp.

#### 2. Non-Clustered Index / Secondary Index (Chỉ mục không cụm)

- **Đặc điểm:** Không can thiệp vào cách dữ liệu thực sự được sắp xếp trên ổ cứng. Nó được tạo ra và nằm ở một nơi **tách biệt hoàn toàn** với cục dữ liệu thật. Giống hệt như cái "Mục lục" ở cuối quyển sách vậy. Do đó, **một bảng có thể có nhiều Non-Clustered Index**.
- **Cấu trúc:** Node lá của Non-Clustered Index KHÔNG chứa dữ liệu thực tế. Thay vào đó, nó chứa một **Con trỏ (Pointer)**.
  - Ở SQL Server: Con trỏ trỏ đến địa chỉ vật lý lưu data.
  - Ở MySQL (InnoDB): Con trỏ chính là giá trị của **Clustered Index (Primary Key)**. (Quá trình này gọi là _Bookmark Lookup_).
- **Tốc độ:** Tốc độ đọc chậm hơn Clustered Index một chút vì phải tốn thêm 1 bước "vòng lại" (Lookup) bằng con trỏ để tìm xuống cục dữ liệu thật. Tuy nhiên nó tiện lợi vì bạn có thể thoái mái tạo mục lục cho các cột thường xuyên bị `WHERE` (như cột `Email` hay `Status`).

**Tóm tắt cho buổi phỏng vấn:**

> _"Clustered Index sắp xếp và chứa trực tiếp dữ liệu thật sự, nên mỗi bảng chỉ được có 1 cái (thường là Primary Key), tìm kiếm cực kì nhanh. Non-Clustered Index nằm tách biệt, chỉ chứa con trỏ trỏ về data thật, nên 1 bảng có thể có nhiều cái, nhưng đọc sẽ chậm hơn do tốn thêm một nhịp Lookup dữ liệu."_

---

### 3. Sự khác biệt giữa `WHERE` và `HAVING`? Thế nào là `GROUP BY`?

Đây là câu hỏi kinh điển về SQL để đánh giá ứng viên có hiểu được thứ tự thực thi (Execution Order) lõi của SQL CSDL hay không.

#### 1. `GROUP BY` là gì?

- Đúng như tên gọi, lệnh `GROUP BY` dùng để **nhóm các dòng (rows)** có chung giá trị ở một hoặc nhiều cột lại thành một dòng duy nhất (Summary row).
- Thường luôn đi kèm với các **Hàm tổng hợp (Aggregate Functions)** như: `COUNT()` (đếm), `SUM()` (tổng), `AVG()` (trung bình), `MAX()`, `MIN()`.
- _Ví dụ:_ Bạn có bảng `Sales` chứa hàng nghìn hoá đơn. Bạn dùng `GROUP BY department_id` kết hợp với `SUM(amount)` để tính ra tổng doanh thu của TỪNG phòng ban thay vì liệt kê từng hoá đơn.

#### 2. Phân biệt `WHERE` và `HAVING`

Cả hai đều đóng vai trò là "Bộ lọc" (Filter) dữ liệu, điều kiện trả ra `TRUE` hoặc `FALSE`. Tuy nhiên, sự khác biệt mấu chốt nằm ở **Thời điểm thực thi**:

| Tiêu chí phân biệt                   | Lệnh `WHERE`                                                                                                                         | Lệnh `HAVING`                                                                                                                                                   |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Bản chất bộ lọc**                  | Lọc dữ liệu trên **từng dòng đơn lẻ (Individual Rows)**.                                                                             | Lọc dữ liệu trên **nhóm các dòng (Grouped Rows)** đã được thiết lập.                                                                                            |
| **Phối hợp với Aggregate Functions** | Tuyệt đối **KHÔNG THỂ** dùng cùng với các hàm như `SUM()`, `COUNT()` vì lúc này dữ liệu chưa được gom nhóm. (Code sẽ văng lỗi ngay). | Sinh ra đặc biệt để **DÙNG CHUNG** trực tiếp với các hàm Aggregate Functions.                                                                                   |
| **Thứ tự thực thi**                  | Chạy **TRƯỚC** khi lệnh `GROUP BY` diễn ra. Dữ liệu mảng rộng sẽ bị gạn lọc bớt thông qua `WHERE` trước khi ném vào rổ gom nhóm.     | Chạy **SAU** khi lệnh `GROUP BY` hoàn tất. Sau khi gom nhóm và tính tính tổng đếm xong, kết quả mới được đưa qua `HAVING` để lọc nốt lần cuối trước khi trả về. |
| **Hiệu suất**                        | Tốt, giúp CSDL loại bỏ bớt dữ liệu rác trước khi phải mệt mỏi xử lý gom nhóm.                                                        | Thường tiêu hao tài nguyên hơn, vì phải chờ gom nhóm/tính toán xong mới bắt đầu lọc dữ liệu lớn.                                                                |

**Bí kíp tóm gọn cho nhà tuyển dụng:**

> _"Mệnh đề `WHERE` được CSDL kích hoạt trước để lọc các dòng dữ liệu đơn lẻ, sau đó mới đến lượt `GROUP BY` gom các dòng thỏa điều kiện lại để tính toán. Cuối cùng, `HAVING` được gọi trễ nhất để lọc các kết quả đã được nhóm/tổng hợp đó trước khi trình trả về Output cuối cùng."_

_Ví dụ minh hoạ sát thực tế:_

```sql
SELECT department_id, SUM(salary) as total_salary
FROM employees
WHERE status = 'Active'         -- Bước 1: Chỉ nhặt những nhân viên đang làm việc (Lọc từng dòng)
GROUP BY department_id          -- Bước 2: Gom nhóm các nhân viên đó theo ID phòng ban và tính SUM
HAVING SUM(salary) > 100000;    -- Bước 3: Chỉ giữ lại các phòng ban có tổng lương trên 100k (Lọc trên tập kết quả nhóm)
```

---

### 4. DBMS là gì? RDBMS khác gì DBMS?

**Khái niệm cơ bản:**
`DBMS` (Database Management System - Hệ quản trị cơ sở dữ liệu) là một phần mềm giúp người dùng giao tiếp, lưu trữ, truy cập và quản lý dữ liệu trong máy tính. Mục đích của nó là thay thế rủi ro khi lưu trữ dữ liệu bằng file text thủ công.
Giữa dòng chảy kiến trúc phần mềm, `RDBMS` (Relational DBMS - Hệ quản trị CSDL Quan hệ) ra đời như một phiên bản tiến hóa tối ưu và phổ biến nhất của DBMS.

**Bảng so sánh cốt lõi khi phỏng vấn:**

| Tiêu chí                       | DBMS truyền thống                                                                                                        | RDBMS (Relational DBMS)                                                                                                                                        |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Bản chất lưu trữ**           | Dữ liệu lưu dưới dạng các **File phẳng (Flat files)** hoặc cấu trúc phân cấp/mạng lưới đơn giản.                         | Dữ liệu tổ chức chặc chẽ dưới dạng các **Bảng (Tables)** bao gồm Hàng (Rows / Records) và Cột (Columns).                                                       |
| **Mối quan hệ (Relationship)** | Các bảng (file) dữ liệu đứng độc lập, **không** thiết lập được mối quan hệ thông minh với nhau.                          | Các bảng có quan hệ móc xích với nhau cực kỳ logic thông qua **Khoá chính (Primary Key)** và **Khoá ngoại (Foreign Key)**.                                     |
| **Chuẩn hoá (Normalization)**  | Không hỗ trợ chuẩn hóa dữ liệu. Dễ dẫn đến dư thừa và rác dữ liệu.                                                       | Hỗ trợ tính chuẩn hóa (Normalization) cực mạnh, đảm bảo dữ liệu luôn duy nhất và không bao giờ bị lặp lại vô nghĩa (Data Redundancy).                          |
| **Tính nhất quán (ACID)**      | Thường lỏng lẻo vật lý, không đảm bảo được nguyên tắc ACID (Tính nguyên tử, Nhất quán, Cô lập, Bền vững) khi server sập. | Tuyệt đối tuân thủ tiêu chuẩn ACID. Đảm bảo Transaction (Giao dịch) an toàn 100% (Ví dụ: Chuyển tiền ngân hàng rớt mạng sẽ tự động Rollback không mất một xu). |
| **Số lượng người dùng**        | Thích hợp cho **Single-user** (chỉ 1 người xử lý dữ liệu cùng lúc, app nhỏ gọn).                                         | Thiết kế tối tân cho **Multiple-users** (Hàng vạn người có thể truy cập sửa xoá cùng lúc bằng luồng cơ chế cấp quyền và Row Locking).                          |
| **Ví dụ phần mềm**             | XML, Windows Registry, MS Access form cũ, FoxPro.                                                                        | MySQL, PostgreSQL, Oracle, Microsoft SQL Server.                                                                                                               |

**Tóm tắt "Bùa hộ mệnh" trả lời nhanh:**

> _"Mọi RDBMS đều là một DBMS, nhưng một DBMS chưa chắc đã là RDBMS. Điểm ăn tiền lớn nhất của chữ R (Relational) nằm ở khả năng liên kết dữ liệu giữa nhiều bảng bằng Khoá ngoại (Foreign Keys), khả năng chuẩn hoá dữ kiện, cơ chế toàn vẹn giao dịch ACID tuyệt đối và được đúc kết hoàn hảo cho hệ thống đa người truy cập đồng thời (Multi-user)."_
