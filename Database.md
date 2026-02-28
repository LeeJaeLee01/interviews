# Cẩm nang Phỏng vấn cơ sở dữ liệu (Database)

Tài liệu này tổng hợp các câu hỏi phỏng vấn thường gặp về Database (CSDL Relational / NoSQL, tối ưu hóa truy vấn).

---

### 1. Indexing (Đánh chỉ mục) là gì? Ưu và Nhược điểm của Index?

**Khái niệm:**
Index (chỉ mục) trong CSDL giống như phần Mục lục ở đầu một quyển sách. Thay vì phải lật từng trang (Full Table Scan) để tìm một thông tin, CSDL sẽ tra cứu trong cấu trúc Index nhỏ gọn trước để tìm ra chính xác "số trang" (con trỏ địa chỉ vật lý) ghi dữ liệu đó. Điều này giúp tăng tốc độ của lệnh `SELECT` (đọc) lên hàng trăm đến hàng ngàn lần.

**Ưu điểm của Index:**

- **Tốc độ:** Tăng tốc độ truy xuất dữ liệu (câu lệnh `SELECT`, `WHERE`) lên rất nhiều lần, đặc biệt là với các bảng có hàng triệu bản ghi.
- **Tối ưu gom nhóm và sắp xếp:** Làm cho các thao tác Sorting (`ORDER BY`) và Grouping (`GROUP BY`) diễn ra mượt mà và ít tốn CPU hơn vì dữ liệu trong Index thường đã được sắp xếp sẵn.
- **Tính toàn vẹn:** Cấu trúc `Unique Index` (như ở Primary Key) vô tình tạo ra một rào chắn cực kỳ vững chắc để đảm bảo không có hai dòng dữ liệu trùng lặp.

**Nhược điểm của Index:**

- **Làm chậm các thao tác Ghi (Write):** Mỗi khi có một thao tác `INSERT`, `UPDATE`, hoặc `DELETE` xảy ra trên bảng, hệ thống CSDL ngoài việc thay đổi dữ liệu thật, còn phải lặn lội tìm và cập nhật lại tất cả các cấu trúc cây Index liên quan (bóp méo cây, phân tách Node...). Do đó, nếu bảng bị "lạm dụng" đánh quá nhiều Index, tốc độ ghi dữ liệu sẽ chậm như rùa bò.
- **Tốn không gian đĩa:** Index là một cấu trúc dữ liệu vật lý riêng biệt, vì vậy việc lưu trữ một đống Index sẽ tốn một lượng dung lượng đĩa bộ nhớ (RAM + Disk space) đôi khi lớn ngang bằng bản thân lượng dữ liệu thật.

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

---

### 5. Khóa chính (Primary Key) và Khóa ngoại (Foreign Key) khác nhau thế nào? Hai khóa này có mặc định đánh Index không?

Đây là cặp khái niệm "xương sống" để thiết lập tính Relational (quan hệ) trong CSDL.

#### 1. Sự khác biệt cơ bản

**Primary Key (Khóa chính):**

- **Mục đích:** Là linh hồn của bảng, dùng để nhận diện và định danh **DUY NHẤT** (Unique) cho mỗi một dòng (Row) trong hệ thống CSDL (như số CCCD của mỗi người).
- **Tính chất:** Tuyệt đối KHÔNG được chứa giá trị `NULL`. Mỗi bảng chỉ được phép có **tối đa 1** Primary Key (Khóa chính này có thể cấu tạo từ một cột đơn là `ID` hoặc nhiều cột ghép lại thành Composite Key).
- **Ví dụ:** Cột `user_id` trong bảng `Users`.

**Foreign Key (Khóa ngoại):**

- **Mục đích:** Là cây cầu kết nối giữa hai bảng. Nó lấy giá trị từ một cột của bảng này (thường là trỏ đến chính Primary Key của bảng kia) để thiết lập **"Mối Quan Hệ" (Relationship)** giữa hai nhóm dữ liệu, đảm bảo không bị mồ côi (Referential Integrity).
- **Tính chất:** CÓ THỂ chứa giá trị `NULL` (nếu liên kết đó không bắt buộc). Một bảng có thể chứa vô số **(từ 0 đến nhiều)** Foreign Key, phụ thuộc vào mức độ phức tạp móc nối của dữ liệu.
- **Ví dụ:** Cột `owner_id` trong bảng `Orders` trỏ về cột `user_id` của bảng `Users` (biết được Order này là của ai).

#### 2. Câu hỏi hóc búa: Hai khóa này có MẶC ĐỊNH được CSDL tự động đánh Index không?

Khẳng định mạnh mẽ trong phòng phỏng vấn: **KHÔNG PHẢI TẤT CẢ ĐỀU ĐƯỢC MẶC ĐỊNH ĐÁNH INDEX!**

**Với Khóa chính (Primary Key):**

- Trả lời là **CÓ**.
- Ở 100% các hệ quản trị CSDL quan hệ (MySQL, PostgreSQL, SQL Server...), cứ hễ bạn khai báo một cấu trúc `PRIMARY KEY`, engine bên dưới sẽ **ngay lập tức tự động** bám một **Clustered Index / Unique Index** (Chỉ mục cụm) vào cột đó. Vì vậy, việc lệnh `SELECT * FROM users WHERE id = 1` luôn có tốc độ tìm kiếm nhanh như điện chớp bẩm sinh mà lập trình viên không cần viết thêm mã `CREATE INDEX`.

**Với Khóa ngoại (Foreign Key):**

- Trả lời là **KHÔNG (mặc định)**.
- Khi bạn cắm cờ `FOREIGN KEY` (ví dụ từ bảng A trỏ sang bảng B), CSDL **chỉ** thiết lập quy tắc ràng buộc "Nếu xoá khoá chính bảng A mà bảng B còn dùng thì không cho xoá" (Ràng buộc toàn vẹn).
- Nó dứt khoát **KHÔNG TỰ ĐỘNG** tạo ra bất cứ mẩu Index (Non-Clustered) nào cho cột Khóa ngoại đó (trong cả Oracle, SQL Server, PostgreSQL...). Ngoại lệ hiếm hoi nằm ở Storage Engine **InnoDB của MySQL**, nó sẽ tự động sinh Index cho Foreign Key nếu bạn chưa tạo.
- **Hệ lụy chí mạng (Cạm bẫy cực lớn của Junior):** Dù khóa ngoại thường xuyên được Dev sử dụng đính vào mẹo câu `JOIN` 2 bảng với nhau. Tuy nhiên, vì CSDL không tự tạo Index nên tốc độ Join hàng triệu dữ liệu sẽ tuột dốc không phanh.
- **Cách xử lý chuẩn Senior:** Khi thiết kế database, dev có kinh nghiệm đều chủ động gõ thêm dòng lệnh `CREATE INDEX index_name ON table_name(foreign_key_column);` thủ công vào cấu trúc schema để tối ưu hóa hiệu năng Query đa bảng.

---

### 6. Làm sao để tối ưu truy vấn (Query Optimization) trong SQL?

Khi tham gia phỏng vấn, "Tối ưu database" là một chủ đề bắt buộc nhà tuyển dụng sẽ hỏi. Để trả lời trọn vẹn, hãy chia cách giải quyết thành 3 phần: Tối ưu câu lệnh, Tối ưu theo chuẩn thực thi (Sequence), và Tối ưu hệ thống lớn.

#### 1. Các thủ thuật viết Query tối ưu hiệu năng (Code Level)

- **Sử dụng Index hợp lý:** Chỉ tạo Index trên các cột thường xuyên xuất hiện trong mệnh đề `WHERE`, `JOIN`, hay `ORDER BY`. Tránh lạm dụng đánh Index vô tội vạ, vì nó sẽ làm các thao tác bóp méo dữ liệu như `INSERT`, `UPDATE` trở nên chậm chạp đi rất nhiều.
- **Tránh dùng `SELECT *`:** Thay vì móc tất cả, hãy chỉ định đích danh từng cột cần thiết (VD: `SELECT id, name, email`). Việc lấy thừa hàng đống cột không bao giờ dùng tới làm lãng phí bộ nhớ RAM, bắt ổ đĩa I/O quay nhiều hơn và làm nghẽn băng thông truyền tải mạng kết nối từ Database tới Server ứng dụng.
- **Ưu tiên `JOIN` thay vì Subquery (Truy vấn lặp):** Các Database Engine đời mới tối ưu quá trình `JOIN` nhiều bảng chung 1 lược đồ thực thi rất tốt. Ngược lại, `Subquery` (truy vấn lồng) đôi khi bị thực thi một cách độc lập nhiều vòng (như vòng lặp O(N^2)) gây tê liệt Server CSDL.
- **Sử dụng `EXPLAIN` (hoặc `EXPLAIN ANALYZE`):** Đây là "bác sĩ siêu âm" của SQL. Bằng cách gõ thêm chữ `EXPLAIN` trước câu Query của bạn, Database sẽ không chạy lệnh đó mà in ra **Query Plan (Kế hoạch thực thi)**: Nó báo cho bạn biết câu lệnh này đang đi theo Index nào, hay đang làm trò rồ dại là Full Table Scan (Quét toàn bảng), và khâu nào đang "ngốn" Cost cao nhất để bạn tối ưu lại.
- **Cẩn thận với dấu `%` ở đầu lệnh `LIKE`:** Nhu cầu tìm kiếm `WHERE name LIKE '%Anh'` (Wildcard ở đầu) sẽ **Vô hiệu hóa toàn bộ B-Tree Index** của cột `name`, vì B-Tree chỉ tra cứu được mốc tính từ trái sang phải. Ép CSDL quay về hình phạt tìm kiếm cực kì chậm.

#### 2. Tối ưu kiến trúc ở Tầng Hệ Thống (Khi dữ liệu khổng lồ)

- **Phân vùng bảng (Partitioning/Sharding):** Nếu một bảng `Orders` có 1 tỷ dòng kéo dài 10 năm, việc Query trong đó là cực hình. CSDL cung cấp cơ chế Partitioning để "chặt" bảng đó ra làm nhiều cục nhỏ (theo Tháng, theo Năm). Khi bạn dùng câu lệnh `SELECT ... WHERE tháng = 1/2024`, CSDL sẽ chỉ vào đúng duy nhất vách chia Partition của tháng 1/2024 để tìm kiếm. Nhanh và tiết kiệm tài nguyên đến kinh ngạc.
- **Sử dụng Caching (Redis/Memcached):** Đối chiếu với quy tắc 80/20. Nếu các dữ liệu Master (như "Danh mục sản phẩm", "Cấu hình Website") bị `SELECT` triệu lần nhưng 1 tuần mới bị `UPDATE` một lần, thì dại gì mà đập liên tục vào Database SQL bắt nó tính lại? Đẩy toàn bộ result đó lên RAM (dùng Redis/Memcached) với TTL (Time-To-Live) để trả lời truy vấn dưới 1ms.

#### 3. Tối ưu dựa trên hiểu biết về "Sequence Query" (Thứ tự thực thi Logic)

Nhiều lập trình viên lầm tưởng một câu lệnh SQL chạy từ trên xuống dưới bắt đầu từ chữ `SELECT`. Hoàn toàn sai! Dưới đây là Thứ tự thực thi logic cốt lõi (Logical Exceution Sequence) của một RDBMS:

> **Thứ tự của RDBMS:** `FROM` & `JOIN` ➜ `WHERE` ➜ `GROUP BY` ➜ `HAVING` ➜ `SELECT` ➜ `ORDER BY` ➜ `LIMIT` / `OFFSET`

**Mẹo tối ưu thần sầu vì hiểu Sequence (Điểm cộng cực mạnh khi Phỏng vấn):**

1.  **Chặn rác từ vòng gửi xe bằng `WHERE`:** Vì mệnh đề `WHERE` được chạy ở ngay Bước 2 (Chỉ đứng sau việc đọc tên Bảng), trong khi `GROUP BY` và `HAVING` chạy tuốt ở giữa. Quy tắc vàng là: Hãy ép một cái phễu `WHERE` vào để gạt bỏ đi 90% dữ liệu không cần thiết ngõ hầu để CSDL tránh việc phải ôm toàn bộ rác đó vào RAM để `GROUP BY` đếm tổng số, rồi mới dùng `HAVING` đá rác ra. Bộ lọc càng sớm, Server CSDL càng nhẹ gánh.
2.  **Tránh bọc Hàm (Function) ở vế trái của `WHERE` (Cạm bẫy SARGable):**
    - ❌ Mã tệ: `WHERE YEAR(create_at) = 2023`
    - ✅ Mã tối ưu chuẩn mực: `WHERE create_at >= '2023-01-01' AND create_at <= '2023-12-31'`
    - **Lý do:** Khi bạn bọc cái cột gốc `create_at` bằng một Hàm số `YEAR()`, CSDL sẽ không bao giờ hiểu được cột đó nữa, nó đành bỏ quên cây B-Tree Index của cột `create_at` và tháo khóa chốt quất Full Table Scan toàn bộ hàng triệu dữ liệu để bóc vỏ hàm `YEAR()` ra rồi mới so sánh.

#### 4. Bài toán nhức nhối khi làm Backend: N+1 Querie (Truy vấn lặp vòng)

- **Vấn đề (Trap N+1):** Căn bệnh kinh điển của các framework ORM (như Laravel Eloquent, Hibernate, TypeORM). Khi bạn viết code (Ví dụ ở Java/PHP/Node) để lấy ra **danh sách 10 Bài Viết** (đây là 1 Query). Lập tức màn hình báo lấy đủ. Nhưng sau đó trong vòng lặp `for` render ra HTML, bạn lại vô tình truy cập `post.Author` để lấy thông tin Tác Giả của từng bài viết đó. Hậu quả là ORM lén lút sinh ra **thêm 10 Queries lẻ tẻ** chĩa vào bảng _Users_ để lấy 10 Tác Giả.
  Tổng cộng hệ thống phải chịu `1 + 10 = 11` Querie đến CSDL. Khối lượng thời gian mạng luân chuyển (Network Round-Trip) cho từng query nhỏ nhoi này gộp lại sẽ bóp chết hoàn toàn tốc độ phản hồi của API.
- **Cách trị:**
  - Sử dụng cơ chế Dùng **Eager Loading** (Ví dụ Laravel là gõ thêm `with('author')`, TypeORM là `relations: ['author']`).
  - Hoặc thuần SQL: Dùng từ khóa `IN (1, 2, 3...)` để gom 10 Authors đó vào nhổ bằng 1 Query duy nhất ngay từ đầu. Giảm số vòng xoay từ N+1 thao tác xuống chỉ còn 2 truy vấn!
