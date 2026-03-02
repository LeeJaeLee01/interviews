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

---

### 7. Deadlock là gì? Xảy ra khi nào và xử lý thế nào?

**1. Deadlock là gì?**
Deadlock (Bế tắc) là tình trạng xảy ra trong cơ sở dữ liệu (hoặc hệ điều hành) khi hai hoặc nhiều giao dịch (transaction) đang chờ đợi lẫn nhau để giải phóng khóa (lock) trên các tài nguyên mà chúng cần để tiếp tục thực thi. Kết quả là không có giao dịch nào có thể tiếp tục và hệ thống bị "treo" ở các giao dịch đó vô thời hạn nếu không có sự can thiệp từ hệ quản trị cơ sở dữ liệu (DBMS).

**Ví dụ thực tế:**

- Giao dịch A và B đang chạy song song.
- Giao dịch A khóa (lock) dòng dữ liệu X và cần khóa dòng Y để hoàn thành.
- Giao dịch B đang giữ khóa dòng Y và lại cần khóa dòng X để hoàn thành.
  => A chờ B nhả Y, B chờ A nhả X. Cả hai tạo ra một vòng lặp chờ đợi chết chóc (Deadlock).

**2. Deadlock xảy ra khi nào?**
Để Deadlock xảy ra, định lý Coffman chỉ ra rằng phải có **đồng thời 4 điều kiện** sau:

1.  **Mutual Exclusion (Độc quyền):** Một tài nguyên (dữ liệu/bảng) tại một thời điểm chỉ được sử dụng bởi một giao dịch duy nhất.
2.  **Hold and Wait (Giữ và Chờ):** Một giao dịch đang giữ ít nhất một tài nguyên và đang chờ đợi thêm tài nguyên khác do giao dịch khác đang giữ.
3.  **No Preemption (Không thể tước đoạt):** Không thể ép buộc lấy lại tài nguyên từ một giao dịch đang giữ nó cho đến khi giao dịch đó tự nguyện giải phóng.
4.  **Circular Wait (Chờ đợi vòng tròn):** Tồn tại một tập hợp các giao dịch (T1, T2, ..., Tn) sao cho T1 đang chờ tài nguyên của T2, T2 chờ của T3,..., và Tn lại chờ tài nguyên của T1.

_Trong Database thực tế_, điều này thường xuyên xảy ra nhất ở các thao tác `UPDATE` chéo nhau trên nhiều bảng dữ liệu trong cùng một Transaction mà không tuân theo một thứ tự chuẩn mực.

**3. Cách xử lý và phòng tránh Deadlock (Dưới góc độ Backend Developer):**

Khi phỏng vấn, nhà tuyển dụng muốn xem cách bạn chủ động "phòng bệnh hơn chữa bệnh".

- **Cách 1: Cập nhật tài nguyên theo cùng một thứ tự (The Golden Rule)**
  Nếu mọi Transaction trong toàn bộ mã nguồn đều luôn luôn truy cập/cập nhật Bảng A rồi mới đến Bảng B, thì Circular Wait sẽ không bao giờ xảy ra. Tiến trình B đến sau sẽ phải đứng xếp hàng chờ A xử lý xong toàn bộ A và B thay vì nhảy vào tranh giành chéo nhau.

- **Cách 2: Giữ cho Transaction thật ngắn gọn (Keep Transactions Short)**
  Giao dịch càng chạy lâu, thời gian nó "ôm" khóa càng dài, nguy cơ đụng độ càng cao. Hãy bóc tách các xử lý logic gọi API ngoài hoặc tính toán nặng ra khỏi khối Transaction của Database. Chỉ gộp những lệnh SQL `INSERT/UPDATE/DELETE` thực sự cần thiết gói gọn vào 1 khối `BEGIN ... COMMIT` chạy nhanh nhất có thể.

- **Cách 3: Chia nhỏ lô dữ liệu (Batching Updates)**
  Thay vì dùng một giao dịch update cùng lúc hàng trăm ngàn dòng dữ liệu (sẽ khóa một lượng tài nguyên khổng lồ rất lâu), hãy chia nhỏ ra thành các lô (batch) 1,000 dòng mỗi lần.

- **Cách 4: Sử dụng độ cô lập (Isolation Level) phù hợp**
  Cân nhắc sử dụng mức độ cô lập thấp hơn như `READ COMMITTED` (mặc định của PostgreSQL và SQL Server) thay vì `SERIALIZABLE` nếu yêu cầu nghiệp vụ cho phép. Mức cô lập càng cao, khóa (lock) càng nghiêm ngặt và giải phóng chậm hơn, dễ sinh ra Deadlock.

- **Cách 5: Bắt lỗi Deadlock và Retry tự động (Cứu cánh cuối cùng)**
  Hầu hết các hệ quản trị CSDL (như InnoDB của MySQL, SQL Server) đều có một **Deadlock Detector (Trình quét tự động)** chạy ngầm. Khi nó phát hiện ra vòng lặp chết chóc, nó sẽ chủ động ra quyết định "giết" (Rollback) một Transaction rẻ tiền nhất (ít chi phí can thiệp nhất) làm nạn nhân (Victim), để vớt giao dịch kia được đi tiếp.
  - _Nhiệm vụ của Code Backend:_ Bạn phải bắt (Catch) được cái Exception/Error do Database ném ra (Ví dụ lỗi code `1213` ở MySQL) và thực hiện cấu hình **Tự động thử lại (Retry)** giao dịch bị hủy đó một vài lần ngầm phía sau (thường kết hợp Exponential Backoff) để user không bị báo lỗi.

**Tóm tắt "Bùa hộ mệnh" trả lời nhanh:**

> _"Deadlock là luồng chờ đợi chéo nhau vô tận giữa 2 Transaction khi tranh chấp tài nguyên. Để giải quyết, ở tầng Code ta phải chuẩn hóa thứ tự truy cập các bảng giống nhau ở mọi nơi, giữ Transaction cực ngắn, chia nhỏ dữ liệu Update và luôn có cơ chế Catch Exception lỗi Deadlock để Retry ngầm cho User."_

---

### 8. Các loại truy vấn trong CSDL (SQL Commands) và cơ chế hoạt động

Trong thế giới CSDL Quan hệ (RDBMS), lệnh SQL được chia thành 5 nhóm chính dựa trên mục đích sử dụng. Khi đi phỏng vấn, mục đích của câu hỏi này thường là kiểm tra xem bạn có phân biệt được thao tác nào tác động vào **cấu trúc** và thao tác nào tác động vào **dữ liệu**, cũng như vì sao lệnh này _Rollback_ được còn lệnh kia thì không.

#### 1. DQL (Data Query Language - Ngôn ngữ truy vấn dữ liệu)

- **Mục đích:** Dùng để đọc và truy xuất dữ liệu từ CSDL mà **KHÔNG** làm thay đổi bất kỳ trạng thái nào của dữ liệu.
- **Lệnh đặc trưng:** `SELECT`.
- **Cơ chế hoạt động:**
  - **Parser:** Kiểm tra cú pháp SQL. Nếu sai báo lỗi liền.
  - **Optimizer:** Phân tích câu lệnh để tìm con đường lấy dữ liệu nhanh nhất (dùng Index nào, Join 2 bảng kiểu Hash hay Nested Loop...) và tạo ra một _Execution Plan (Kế hoạch thực thi)_.
  - **Executor:** Dựa trên Execution Plan, lấy dữ liệu từ đĩa hoặc Buffer Cache trên RAM, xử lý `WHERE`, `GROUP BY`, `ORDER BY` rồi trả kết quả cho Client.

#### 2. DDL (Data Definition Language - Ngôn ngữ định nghĩa dữ liệu)

- **Mục đích:** Tác động vào **Cấu trúc (Schema / Metadata)** của CSDL. Dùng để tạo mới, thay đổi, hoặc hủy bỏ các đối tượng như Database, Bảng (Table), Chỉ mục (Index), View...
- **Lệnh đặc trưng:** `CREATE` (tạo), `ALTER` (sửa cấu trúc), `DROP` (xóa cấu trúc), `TRUNCATE` (xóa sạch dữ liệu nhưng reset bộ đếm ID, bản chất là drop bảng rồi tạo lại bảng mới cực nhanh).
- **Cơ chế hoạt động (Chú ý mạnh):**
  - DDL thay đổi các tệp cấu trúc hệ thống (Data Dictionary).
  - Hầu hết trong các hệ quản trị (như Oracle, MySQL), lệnh DDL có tính chất **Auto-commit (Tự động lưu ngay lập tức)**. Do đó, một khi đã chạy `DROP TABLE` là cấu trúc bị phá hủy ngay, KHÔNG THỂ gọi lệnh `ROLLBACK` để hoàn tác lại như chỉnh sửa dữ liệu được. (Ngoại trừ PostgreSQL hỗ trợ Transactional DDL).

#### 3. DML (Data Manipulation Language - Ngôn ngữ thao tác dữ liệu)

- **Mục đích:** Tác động sát ván vào **Nội dung / Dữ liệu** bên trong các hàng (row) của bảng.
- **Lệnh đặc trưng:** `INSERT` (thêm), `UPDATE` (sửa), `DELETE` (xóa hàng).
- **Cơ chế hoạt động:**
  - Khi chạy (`UPDATE`/`DELETE`), dữ liệu **không** được ghi đè trực tiếp xuống ổ cứng vật lý ngay lập tức (vì đọc ghi đĩa HDD/SSD rất chậm).
  - CSDL tải (load) trang dữ liệu đó lên RAM (Buffer Pool), thay đổi dữ liệu trên RAM.
  - ĐỒNG THỜI, nó sẽ ghi nhanh một dòng **nhật ký (Log)** vào file _Write-Ahead Log (WAL) / Transaction Log_ lưu trên đĩa để phòng hờ bị rút phích cắm điện.
  - Về sau, một trình dọn dẹp chạy ngầm mới từ từ ghi các thay đổi từ RAM xuống file dữ liệu thật.
  - Lệnh DML **Không tự động Commit**. Bạn bắt buộc phải gọi `COMMIT` thì mới có tác dụng thật, hoặc gọi `ROLLBACK` để Undo nó nếu gõ nhầm.

#### 4. DCL (Data Control Language - Ngôn ngữ điều khiển dữ liệu)

- **Mục đích:** Quản lý an ninh, phân quyền và cấp tài khoản cho người dùng (Users) truy cập vào DB. Nó là xương sống của cơ chế Multi-User.
- **Lệnh đặc trưng:** `GRANT` (Cấp quyền, ví dụ cấp quyền _chỉ được SELECT_ cho nhân viên tập sự), `REVOKE` (Thu hồi quyền).
- **Cơ chế hoạt động:** Thay đổi các bảng quy định về User Permission (như bảng `mysql.user`). Bất cứ Request nào bay vào CSDL cũng phải đi qua bước rà soát phân quyền này trước khi tới Parser.

#### 5. TCL (Transaction Control Language - Ngôn ngữ điều khiển giao dịch)

- **Mục đích:** Quản lý các block **DML**, gộp nhiều thao tác lại để đảm bảo tính ACID (Toàn vẹn dữ liệu). Hoặc là Thành công tất cả, hoặc Thất bại thì không lưu phần nào.
- **Lệnh đặc trưng:** `COMMIT` (Lưu chính thức), `ROLLBACK` (Hoàn tác và xóa sạch thay đổi), `SAVEPOINT` (Đánh dấu mốc giữa chừng để lỡ Rollback thì Rollback về mốc đó).
- **Cơ chế hoạt động:**
  - Lợi dụng vùng **Undo Log** trên CSDL. Khi code gọi `ROLLBACK`, hệ thống sẽ đọc ngược cái _Undo Log_ đó để áp dụng các thao tác nghịch đảo (Ví dụ bạn vừa `INSERT` thì nó ngầm `DELETE` lại) nhằm xóa dấu vết trên RAM.

**Tóm tắt "Bùa hộ mệnh" cho phòng phỏng vấn:**

> _"SQL chia 5 loại chính: **DQL** (`SELECT`) dùng để đọc. **DDL** (`CREATE, DROP`) dùng để tác động lên **Cấu trúc bảng**, chạy xong là lưu ngay (Auto-Commit) không thể Rollback cứu vãn. **DML** (`INSERT, UPDATE, DELETE`) dùng để sửa **Dữ liệu thực** trên RAM rồi ghi vào Log ẩn, có thể Rollback trước khi chốt đơn. **DCL** (`GRANT`) để phân quyền sinh tử. Cuối cùng **TCL** (`COMMIT, ROLLBACK`) chính là nhạc trưởng điều phối các lệnh DML gom vào 1 Transaction vòng nguyên tử (ACID)."_

#### 6. Transaction trong Database — gồm những phần nào, hoạt động thế nào?

**Transaction (Giao dịch)** là một đơn vị công việc logic gồm **nhiều thao tác** (thường là các lệnh DML) được xử lý **một khối**: hoặc tất cả thành công, hoặc tất cả bị hủy. DBMS đảm bảo tính **ACID** cho mỗi transaction.

**Các phần (thành phần) của một Transaction:**

1. **Bắt đầu transaction (Begin)**
   - Ngầm định: mỗi lệnh DML chạy xong có thể được gom vào một transaction cho đến khi gọi COMMIT/ROLLBACK (chế độ auto-commit tắt) hoặc mỗi lệnh tự commit (auto-commit bật).
   - Tường minh: `BEGIN;` / `START TRANSACTION;` — đánh dấu điểm bắt đầu, mọi thay đổi sau đó thuộc cùng một transaction.

2. **Các thao tác (Operations)**
   - Các lệnh **DML** trong transaction: `INSERT`, `UPDATE`, `DELETE`.
   - Kết quả tạm thời chỉ phản ánh trên **bộ nhớ (Buffer Pool)** và ghi vào **Transaction Log (WAL)**; chưa “chốt” vào dữ liệu chính thức cho đến khi COMMIT.

3. **Savepoint (tùy chọn)**
   - `SAVEPOINT tên;` — đánh dấu một mốc giữa chừng.
   - `ROLLBACK TO SAVEPOINT tên;` — hủy mọi thay đổi sau mốc đó nhưng vẫn giữ transaction (và các thay đổi trước savepoint). Dùng khi muốn thử một nhánh logic rồi quay lại mà không mất toàn bộ transaction.

4. **Kết thúc transaction — COMMIT hoặc ROLLBACK**
   - **COMMIT**: Xác nhận ghi vĩnh viễn. DBMS “chốt” thay đổi: flush log, cập nhật dữ liệu, giải phóng khóa (lock), transaction kết thúc.
   - **ROLLBACK**: Hủy bỏ toàn bộ thay đổi của transaction. DBMS dùng **Undo Log** (và có thể Redo/Undo) để khôi phục trạng thái trước khi transaction bắt đầu, giải phóng khóa, transaction kết thúc.

**Cơ chế hoạt động (tóm tắt):**

- **Atomicity (Tính nguyên tử):** Nhờ **Transaction Log (WAL)** và **Undo Log**. Mọi thay đổi được ghi log trước; khi COMMIT thì log được flush và coi là “đã commit”. Khi ROLLBACK thì áp dụng ngược (undo) theo log để trả dữ liệu về trạng thái cũ.
- **Consistency (Nhất quán):** Ràng buộc (constraint, trigger) được kiểm tra trong transaction; nếu vi phạm thì transaction fail và có thể rollback. Commit chỉ thành công khi dữ liệu thỏa mọi ràng buộc.
- **Isolation (Cô lập):** Mỗi transaction “thấy” dữ liệu theo mức isolation (READ UNCOMMITTED → READ COMMITTED → REPEATABLE READ → SERIALIZABLE). DBMS dùng **Lock** (hàng, bảng, gap) và **MVCC** (Multi-Version Concurrency Control) để tránh dirty read, lost update, phantom read tùy level.
- **Durability (Bền vững):** Sau khi COMMIT, thay đổi đã ghi vào log trên đĩa; dù DB crash thì khi khởi động lại vẫn **replay log** để khôi phục dữ liệu đã commit.

**Tóm tắt trả lời phỏng vấn:**

> _Transaction gồm: (1) Bắt đầu (BEGIN/START TRANSACTION), (2) Các thao tác DML, (3) Tùy chọn SAVEPOINT, (4) Kết thúc bằng COMMIT (lưu vĩnh viễn) hoặc ROLLBACK (hủy toàn bộ). DBMS đảm bảo ACID nhờ Transaction Log, Undo Log, lock và MVCC: thay đổi ghi log trước, COMMIT thì chốt và flush log, ROLLBACK thì undo theo log; isolation điều khiển mức “nhìn thấy” dữ liệu giữa các transaction đồng thời._

---

### 9. Câu hỏi thực tế: Khi chạy lệnh SELECT chứa JOIN và Phân trang (LIMIT / OFFSET), CSDL thực thi như thế nào và rủi ro là gì?

Đây là một câu hỏi phân loại ứng viên Senior/Junior cực kỳ phổ biến. Nhà tuyển dụng muốn kiểm tra xem bạn có nắm được rủi ro gây sập hệ thống có tên là **Deep Pagination (Phân trang sâu)** hay không.

#### 1. Cơ chế thực thi mặc định của Database

Giả sử bạn có truy vấn:
`SELECT * FROM Orders O JOIN Users U ON O.user_id = U.id WHERE O.status = 'Paid' ORDER BY O.created_at DESC LIMIT 10 OFFSET 10000;`

Database sẽ xử lý theo trình tự vô cùng cồng kềnh như sau:

1. **Lọc và JOIN:** Hệ thống duyệt bảng `Orders` tìm các đơn trạng thái `Paid`. Sau đó, nó thực hiện móc nối (JOIN) khối dữ liệu này với bảng `Users` trên RAM/Disk.
2. **Sắp xếp (ORDER BY):** Toàn bộ khối dữ liệu vừa JOIN xong sẽ được đưa vào bộ nhớ để sắp xếp giảm dần theo thời gian.
3. **Cắt Pagination (OFFSET / LIMIT):** Cuối cùng, CSDL đếm từ trên xuống dưới, **bỏ đi (skip)** đúng số dòng của `OFFSET` (10,000 dòng). Đáng buồn là 10,000 dòng này trước đó đã tiêu tốn tài nguyên khổng lồ để đọc mạng, ráp JOIN và sắp xếp nhọc nhằn, nhưng chốt lại bị VỨT VÀO SỌT RÁC. Nó chỉ giữ lại 10 dòng trúng thưởng tiếp theo thuộc phạm vi `LIMIT` để trả về cho Client.

#### 2. Rủi ro chết người (Deep Pagination)

- Khi User lật trang càng sâu (VD: `LIMIT 10 OFFSET 10,000,000`), thời gian phản hồi càng chậm lên theo cấp số nhân (độ phức tạp **O(N)**).
- CSDL phải quét, móc nối và sắp xếp ròng rã **10,000,010** dòng, rồi thẳng tay xóa nợ 10 triệu dòng đầu tiên. Quá trình này ăn sạch RAM (OOM), thắt nút cổ chai I/O Disk và kéo sập CPU gây Time out hoàn toàn API.

#### 3. Cách trả lời ghi điểm tuyệt đối (Khắc phục Deep Pagination)

Tuyệt đối đừng chỉ dừng lại ở việc nêu nhược điểm, hãy chốt sale bằng 3 phương pháp giải quyết sau:

- **Cách 1: Kỹ thuật "Subquery phân trang trước" (Deferred Join / JOIN trì hoãn)**
  Thay vì móc nối `JOIN` nguyên 2 cái bảng bự nái ngay từ đầu dẫn tới phân trang chậm, ta lợi dụng Subquery lên _riêng bảng gốc_ để giới hạn đủ mốc 10 dòng (IDs) đó trước.
  ```sql
  SELECT * FROM Orders O
  JOIN (
      SELECT id FROM Orders WHERE status = 'Paid' ORDER BY created_at DESC LIMIT 10 OFFSET 10000
  ) SubO ON O.id = SubO.id
  JOIN Users U ON O.user_id = U.id;
  ```
  Ta lọc thuần trên bảng Order ăn thẳng vào Index với tốc độ ánh sáng. Lấy được đúng 10 cái ID mục tiêu rồi, ta mới móc 10 ID đó đem đi JOIN vỏn vẹn 10 vòng với các thông tin thịt còn lại của bảng Users. Nhanh hơn gấp trăm lần!
- **Cách 2: Keyset Pagination (Cursor Pagination / Seek Method)**
  Tẩy chay hoàn toàn từ khóa `OFFSET`. Thay vì bắt CSDL đếm mù quáng dòng thứ mấy, ta truyền vào tham số mốc (Ví dụ: ID của đơn hàng cuối cùng ở trang trước là `999`).
  `SELECT * FROM Orders WHERE id > 999 AND status = 'Paid' ORDER BY id ASC LIMIT 10`
  Cách này chạy trên Index B-Tree với uy lực O(1). Khách bấm trang 1 hay trang 1 triệu thì thời gian phản hồi vẫn là 1 milisecond. Nhược điểm là UI chỉ thiết kế được nút (Xem tiếp / Trở lại), không nhảy vọt thẳng đến trang số 50 được.

- **Cách 3: Giới hạn nghiệp vụ (Business Rule)**
  Hãy khuyên Product Manager rằng đến cả Google cũng chặn User không cho lướt tới trang thứ 1.000. Ta giới hạn cứng API chỉ cho phép `OFFSET 1.000`. Nếu muốn tìm sâu hơn, hãy khuyến khích User chủ động dùng **Thanh công cụ lọc (Filter tool)** thay vì cuộn mù quáng.

- **Tip bổ sung "Dùng DISTINCT ON" trong bài toán phân trang JOIN 1-N (Đặc sản PostgreSQL):**
  Trong thực tế, khi bạn phân trang JOIN bảng `Users` (1) với `Orders` (N) để lấy ra **"Đơn hàng cập nhật mới nhất của mỗi người dùng ở trang hiện tại"**, câu JOIN sẽ sinh ra các dòng lặp (duplicate rows của 1 người có 100 đơn), làm hỏng bộ đếm `LIMIT/OFFSET`.
  Lúc này, thay vì dùng `GROUP BY` lềnh kềnh gây kẹt CPU, ta dùng lệnh `DISTINCT ON` nội bộ trong một Subquery trước để triệt tiêu các bản ghi trùng lặp đi, thu lại duy nhất 1 "Đơn hàng mới nhất của 1 người" rồi mới mang đi phân trang + JOIN.
  ```sql
  SELECT * FROM (
      SELECT DISTINCT ON (user_id) *
      FROM Orders
      ORDER BY user_id, created_at DESC
  ) LatestOrders
  JOIN Users U ON LatestOrders.user_id = U.id
  LIMIT 10 OFFSET 0;
  ```

**Tóm tắt "Bùa hộ mệnh" cho câu hỏi này:**

> _"Câu truy vấn SELECT JOIN kèm OFFSET lớn sẽ ép CSDL quét rác từ đầu chí cuối tốn cực nhiều tài nguyên rồi vứt đi phần OFFSET, sinh ra bế tắc Deep Pagination. Giải pháp là chuyển sang kỹ thuật **Subquery Phân trang trước (Deferred Join)**, hoặc dùng **Keyset Pagination** (so sánh con trỏ trực tiếp bằng `WHERE id > last` với sức mạnh O(1)), hoặc giới hạn Max Offset. Ngoài ra nếu JOIN bị trùng lặp 1-N, hãy bọc một Subquery **DISTINCT ON** để loại bỏ mảng lặp trước khi đếm LIMIT phân trang."_

---

### 10. Sự khác biệt cốt lõi giữa `DISTINCT` và `DISTINCT ON`

Trong khi `DISTINCT` là cú pháp tiêu chuẩn có mặt ở mọi Hệ quản trị CSDL Quan hệ (MySQL, SQL Server...), thì `DISTINCT ON` lại là một **"đặc sản" siêu năng lực chỉ có riêng trên hệ quản trị PostgreSQL**.

Điểm khác biệt lớn nhất nằm ở cách chúng định nghĩa từ "Trùng lặp". Cụ thể:

#### 1. `DISTINCT`: Bộ lọc cứng nhắc (Xét trùng trên TẤT CẢ các cột hiện hình)

- **Cơ chế:** Lệnh `DISTINCT` sẽ soi xét **toàn bộ** các cột mà bạn khai báo đằng sau chữ `SELECT`. Nếu có 2 dòng dữ liệu mà giống y đúc nhau 100% ở _từng giá trị của mọi cột đó_, nó mới coi là lặp và gạch bỏ đi 1 dòng.
- **Điểm yếu:** Chỉ cần 1 cột bị móp méo/khác nhau (ví dụ ID hoặc Thời gian), `DISTINCT` sẽ câm nín và coi đó là 2 bản ghi khác biệt hoàn toàn (không thèm lọc).

**Ví dụ thực tế:**
Bảng `Purchases` có 3 dòng:

1. `user_id: 1` | `item: Apple` | `date: 2024-01-01`
2. `user_id: 1` | `item: Apple` | `date: 2024-01-01`
3. `user_id: 1` | `item: Banana` | `date: 2024-01-02`

```sql
-- TRUY VẤN:
SELECT DISTINCT user_id, item FROM Purchases;

-- KẾT QUẢ: (Nó tự động trộn 2 cái Apple lại làm 1)
1 | Apple
1 | Banana
```

#### 2. `DISTINCT ON`: Bộ lọc linh hoạt (Chỉ xét trùng trên MỘT VÀI CỘT chủ đích)

- **Cơ chế:** `DISTINCT ON (...cột...)` cho phép bạn chỉ định rõ ràng: _"Ê PostgreSQL, tính từ bây giờ tao chỉ xét trùng lặp dựa trên 1 cột (hoặc 1 cụm cột) cụ thể mà thôi. Còn các cột dữ liệu râu ria kéo theo bên cạnh, tao không quan tâm chúng nó giống hay khác nhau!"_.
- PostgreSQL sẽ bóc tách dư liệu thành các nhóm (Groups) dựa theo tiêu chí đó. Ở mỗi nhóm, nó **chỉ chắt lọc ra đúng 1 dòng đầu tiên** nó đụng mặt, và nhẫn tâm ném bỏ tất cả các bản ghi còn lại.
- **Luật bất thành văn (Dùng chung với `ORDER BY`):** Vì nó chỉ trích xuất "Dòng đầu tiên" gặp được, nên bạn BẮT BUỘC phải xài `ORDER BY` phía sau để chủ đích điều hướng cái "Dòng đầu tiên" đó là dòng Tốt nhất / Mới nhất / Cao điểm nhất cho nhóm đó.

**Ví dụ thực tế siêu kinh điển (The "Latest row per group" problem):**
Bảng `Orders` ghi lại lịch sử mua hàng, 1 khách (user_id = 1) có thể mua nhiều lần.

1. `id: 101` | `user_id: 1` | `total: 50$` | `created_at: 08:00 AM`
2. `id: 102` | `user_id: 1` | `total: 200$` | `created_at: 09:00 AM` (Mới nhất của User 1)
3. `id: 103` | `user_id: 2` | `total: 10$` | `created_at: 10:00 AM` (Mới nhất của User 2)

**Bài toán:** Hãy xuất ra **Đơn hàng mới mua gần đây nhất của TỪNG người dùng**, hiển thị kèm Cột ID và Cột Tổng Tiền.

- **Bi kịch của DISTINCT:** Nếu bạn dùng `SELECT DISTINCT(user_id) id, total`, nó sẽ in ra cả dòng 101 và 102. Lý do là vì ID 101 khác 102, tổng tiền 50 khác 200, nên `DISTINCT` đầu hàng.
- **Sức mạnh của DISTINCT ON:**

```sql
SELECT DISTINCT ON (user_id) id, user_id, total, created_at
FROM Orders
ORDER BY user_id, created_at DESC; -- Phải sắp xếp Thời gian giảm dần để "dòng đầu tiên" luôn rơi vào dòng MỚI NHẤT
```

-- KẾT QUẢ TRẢ VỀ:
-- Nó gộp nhóm theo user_id, thấy user 1 có 2 đơn, nó chỉ bốc dòng trên cùng (nhờ ORDER DESC) là dòng 102 và hủy dòng cũ 101.
102 | 1 | 200$ | 09:00 AM
103 | 2 | 10$ | 10:00 AM

```
**Tóm tắt "Bùa hộ mệnh" khác biệt:**
> *"**DISTINCT** truyền thống rất cứng nhắc, nó chỉ chịu lọc trùng khi tất cả các cột bạn Select ra đều trông giống y đúc nhau 100%. Trái ngược hoàn toàn, **DISTINCT ON** (của Postgres) là vũ khí hạng nặng để giải quyết bài toán nhức nhối 'Lấy thông tin bản ghi MỚI NHẤT của TỪNG đối tượng' bằng cách chỉ phân định trùng lặp trên 1 Cột gom nhóm, kết hợp mượt mà cùng ORDER BY để tự động lựa rút ra đúng 1 dòng ngon nhất trồi lên và thủ tiêu các bản sao cũ."*
```

---

### 11. Tổng hợp các loại JOIN trong CSDL, cơ chế hoạt động và ví dụ thực tế

`JOIN` là kỹ năng cơ bản nhất để kết nối "những mảnh ghép" dữ liệu phân tán ở kiến trúc CSDL Quan hệ (Relational Database).

Để dễ mường tượng, ta lấy chung một kịch bản dữ liệu sau:

- **Bảng `Users` (Người dùng):** ID = 1 (Alice), ID = 2 (Bob), ID = 3 (Charlie).
- **Bảng `Orders` (Đơn hàng):** user_id = 1 (mua Gà rán), user_id = 2 (mua Trà sữa), user_id = 4 (Khách vãng lai, mua Bánh mì).

_(Nhận thấy: Charlie không có mua gì. Và Đơn Bánh Mì không thuộc về User nào cả trong hệ thống)._

#### 1. INNER JOIN (Giao điểm chung - Lấy phần giao nhau)

- **Cơ chế:** Hoạt động như phép "Giao" (Intersection) trong Toán học tổ hợp. Nó chỉ lọc ra và ghép nối các bản ghi mà **Khóa liên kết (JOIN KEY) phải xuất hiện đồng thời ở CẢ HAI bảng**. Những bản ghi "mồ côi" (lệch pha) ở 1 trong 2 bảng sẽ bị bỏ qua.
- **Khi nào dùng:** Khi bạn chỉ muốn lấy những Khách hàng **ĐÃ MUA** đơn, không quan tâm khách ảo không mua.
- **Ví dụ Query:**

```sql
SELECT U.name, O.item
FROM Users U
INNER JOIN Orders O ON U.id = O.user_id;

-- KẾT QUẢ: (Chỉ xuất hiện Alice và Bob. Charlie và Đơn Bánh mì bị loại vì không có điểm chung trượt khớp).
Alice | Gà rán
Bob   | Trà sữa
```

#### 2. LEFT JOIN (Hoặc LEFT OUTER JOIN - Lấy trọn bộ bảng TRÁI)

- **Cơ chế:** Nó bốc **TOÀN BỘ** dữ liệu của Bảng bên Trái (Bảng đứng trước chữ JOIN) làm gốc. Sau đó nó tìm các mảnh ghép tương ứng ở Bảng bên Phải đắp vào.
- _Điều quan trọng:_ Nếu dòng bên Trái không tìm thấy dòng khớp bên Phải, nó vẫn giữ lại dòng bên Trái đó và **điền chữ `NULL`** cho các cột bên Phải.
- **Khi nào dùng:** Cực kỳ quan trọng để làm báo cáo, ví dụ: Lấy ra "Tổng danh sách toàn bộ khách hàng" và kèm theo đơn họ mua (Nếu ai chưa mua thì để trống `NULL`).
- **Ví dụ Query:**

```sql
SELECT U.name, O.item
FROM Users U       -- Bảng Trái là gốc
LEFT JOIN Orders O ON U.id = O.user_id;

-- KẾT QUẢ: (Charlie xuất hiện dù ất ơ không mua gì, Đơn Bánh mì của ID=4 vô danh bị ném bỏ).
Alice   | Gà rán
Bob     | Trà sữa
Charlie | NULL
```

#### 3. RIGHT JOIN (Hoặc RIGHT OUTER JOIN - Lấy trọn bộ bảng PHẢI)

- **Cơ chế:** Ngược lại hoàn toàn với LEFT JOIN. Nó ưu tiên lấy cứng **TOÀN BỘ** dữ liệu bảng bên Phải (Bảng đứng sau chữ JOIN) làm gốc. Cái nào bảng Trái không có thì điền thẻ `NULL`.
- **Mẹo thực tế:** Rất ít dev xài `RIGHT JOIN`, vì người ta có thói quen đảo ngược vị trí hai bảng lại và dùng `LEFT JOIN` cho dễ đọc code não trái sang não phải.
- **Ví dụ Query:**

```sql
SELECT U.name, O.item
FROM Users U
RIGHT JOIN Orders O ON U.id = O.user_id; -- Bảng Phải (Orders) là gốc

-- KẾT QUẢ: (Xuất hiện Đơn Bánh mì vô danh, Charlie bị vứt bỏ vì không có đơn).
Alice | Gà rán
Bob   | Trà sữa
NULL  | Bánh mì
```

#### 4. FULL JOIN (Hoặc FULL OUTER JOIN - Lấy Sạch Mọi Thứ)

- **Cơ chế:** Phép "Hợp" (Union) trong Toán học. Bốc toàn bộ các dòng của bảng A và TOÀN BỘ dòng bảng B ghép lại. Thằng nào khớp thì nằm chung hàng, thằng nào dư bảng A thì bên B NULL, dư bảng B thì bên A NULL. (Sẽ sinh ra kết quả lớn nhất).
- **Lưu ý:** MySQL mặc định **không hỗ trợ** dòng code `FULL JOIN`. Bạn phải tự làm trò UNION cái `LEFT JOIN` và `RIGHT JOIN` lại với nhau. PostgreSQL thì chạy phà phà.
- **Ví dụ Query:**

```sql
SELECT U.name, O.item
FROM Users U
FULL JOIN Orders O ON U.id = O.user_id;

-- KẾT QUẢ: Lấy tất cả khách hàng, lấy tất cả đơn vãng lai. Trống đâu điền NULL đó.
Alice   | Gà rán
Bob     | Trà sữa
Charlie | NULL
NULL    | Bánh mì
```

#### 5. SELF JOIN (Tự lấy mỡ nó rán nó)

- **Cơ chế:** Là hành động kỳ dị khi lấy **1 Bảng JOIN lại với chính nó**.
- **Khi nào dùng:** Xử lý cấu trúc dữ liệu phả hệ/Cây thư mục, ví dụ Bảng `Employees` chứa dòng `Sếp` và `Nhân Viên`, nhân viên có cột `manager_id` trỏ lại về `id` của Sếp ngay trong cùng bảng đó.
- **Nhận diện:** Bắt buộc đặt tên tắt rạch ròi 2 cái Alias biến thể để phân biệt vai diễn. Bảng 1 vai "Nhân Viên" `E`, Bảng 2 sắm vai "Ông Sếp" `M`.
- **Ví dụ Query:**

```sql
SELECT E.name AS NhanVien, M.name AS TruongPhong
FROM Employees E
LEFT JOIN Employees M ON E.manager_id = M.id;
```

---

#### 🔧 [Deep Dive] Cơ chế Toán học BÊN DƯỚI các hàm JOIN (Query Engines)

Để thể hiện level Senior/Mid trước nhà tuyển dụng, hãy nói qua việc CSDL dùng ngầm thuật toán gì để móc lối 2 bảng với nhau ở tầng đĩa vật lý:

1. **Nested Loop Join (Vòng lặp thần chưởng O(N\*M))**
   - _Cách chạy:_ For vòng lặp từng dòng ngoài cùng ở Bảng ngoài (Outer table), vác nó đi soi so sánh với TẤT CẢ các dòng ở bảng bên trong (Inner table). Siêu siêu chậm nếu không đánh index.
2. **Hash Join (Dùng bảng Băm trên RAM, cực nhanh cho toán hạng bằng `=`)**
   - _Cách chạy:_ Nó dồn hết Hash ID các dòng bảng A vào 1 cái HashMap memory trên RAM. Cầm từng dòng bảng B chọi vô Hash Table tìm key. Complexity tiệm cận O(N). RDBMS đời mới cực kỳ thích chiêu này.
3. **Merge Join (Yêu cầu phải xếp trật tự trước)**
   - _Cách chạy:_ Cả 2 bảng đều được ORDER BY sắp xếp trước, sau đó nó kẹp con trỏ trượt rẹt rẹt 1 đường giống như kéo phéc-mơ-tuya. Thường xảy ra tự động nếu 2 bảng móc JOIN qua Primary Key có sẵn Clustered Index.

**Tóm tắt "Bùa hộ mệnh" trả lời phỏng vấn:**

> _"Về kiến trúc logic, JOIN chia thành 4 loại cơ bản: **INNER** (Chỉ lấy giao điểm chuẩn khớp), **LEFT/RIGHT** (Thiên vị lấy toàn bộ 1 bên làm Gốc, vế kia lệch thì bôi NULL bù vào) và **FULL** (Thầu tất cả). Ngoài ra có thể **SELF JOIN** để kết nối vòng với chính nó (Phả hệ cậy thư mục). Về tầng thực thi vật lý, CSDL hay dùng 3 thuật toán là **Nested Loop** (Dùng cho 2 bảng cực nhỏ), **Hash Join** (Tạo map băm memory để xử lý bảng lớn cực nhanh) hoặc **Merge Join** (quét con trỏ nếu 2 cây Index tham gia đã được đánh sẵn Clustered)."_

---

### 12. Cách đánh Index cho kiểu dữ liệu JSON / JSONB (Đặc sản PostgreSQL)

Trong các hệ CSDL quan hệ hiện đại (đặc biệt là PostgreSQL), việc nhồi dữ liệu phi cấu trúc vào một cột `JSONB` đang trở thành "bảo bối" của các Backend Developer để giải quyết các lược đồ (schema) thay đổi liên tục.
Tuy nhiên, nếu bạn cứ query chay (chọc sâu vào JSON để tìm kiếm) mà không đánh Index, thì Full Table Scan sẽ kéo nghẽn sập hệ thống ngay.

Câu hỏi này dùng để nhà tuyển dụng xem bạn có thực sự "thực chiến" với JSON trong DB SQL hay chỉ dừng ở mức biết cú pháp.

Sau đây là 3 chiến thuật đánh Index cho JSONB phổ biến nhất:

#### 1. Dùng GIN Index (Generalized Inverted Index) nguyên bản — Cân mọi thể loại

- **Bản chất:** GIN sinh ra để làm "Mục lục từ khóa" (Inverted Index) chuyên trị các kiểu dữ liệu mảng (Arrays) hoặc Document phức tạp như JSONB. Nó rã toàn bộ cấu trúc JSON ra thành từng mảnh Node (cả Key lẫn Value) để ánh xạ điểm trỏ.
- **Khi nào dùng:** Khi ứng dụng của bạn có nhu cầu tìm kiếm **linh hoạt**, không lường trước được sẽ chọc vào Key nào, và xài nhiều toán tử dò dẫm như `?` (Chứa key không?), `@>` (Có chứa cục JSON con này khồng?).
- **Cú pháp:**
  ```sql
  CREATE INDEX idx_users_metadata ON Users USING GIN (metadata);
  ```
- **Query ăn Index cực mạnh:**
  ```sql
  -- Truy tìm khách hàng nào có sở thích 'Coding' nằm trong JSON Array
  SELECT * FROM Users WHERE metadata @> '{"hobbies": ["Coding"]}';
  ```
- **Nhược điểm:** Tốn RẤT RẤT NHIỀU dung lượng ổ cứng (ví dụ cục JSON 1MB thì Index của nó phình ra kinh dị), cộng thêm làm thao tác `INSERT/UPDATE` chậm đứt dây thần kinh vì phải xây dựng lại cây băm khổng lồ. Và nó KHÔNG TRỢ GIÚP cho việc sắp xếp `ORDER BY`.

#### 2. Dùng GIN Index với cờ `jsonb_path_ops` — Vũ khí bí mật siêu nhẹ

- **Bản chất:** Phiên bản tối ưu của GIN mặc định. Thay vì rã nát từng key/value riêng lẻ, nó lại đi băm mã Hash của **CẢ CÁI ĐƯỜNG DẪN** (Path) từ Key nối thẳng đến Value lại làm 1.
- **Ưu điểm:** Kích thước Index thu nhỏ đi 1/3 hoặc 1/2 so với GIN thường. Tốc độ tìm bằng toán tử Containment `@>` (Chứa đoạn JSON này không) lại NHANH HƠN ĐÁNG KỂ do so sánh Hash.
- **Nhược điểm "Chí mạng":** Nó bị "phế võ công" ở toán tử dấu hỏi `?` (Hỏi xem Key này có tồn tại không). Bạn chỉ được dùng duy nhất toán tử `@>`.
- **Cú pháp:**
  ```sql
  CREATE INDEX idx_users_metadata_path ON Users USING GIN (metadata jsonb_path_ops);
  ```

#### 3. Trích xuất ra Text rồi đánh B-Tree Index truyền thống — 1 phát ăn trọn

- **Bản chất:** Nếu bạn nắm thóp được yêu cầu Business rằng ứng dụng **chỉ luôn luôn tìm kiếm đúng 1 trường duy nhất** nằm sâu thẳm trong JSON (Ví dụ: Tìm theo `email` nằm trong chuỗi `attributes`). Thì dại gì mà đem cả cục JSON mập thù lù đó đi ép thành GIN Index cho cồng kềnh? Đơn giản, hãy trích đúng cái Email đó ra thành văn bản thuần, và úp sọt cho nó cái B-Tree Index cổ điển.
- **Ưu điểm:** Nhẹ tựa lông hồng, nhanh như chớp, cập nhật tốn ít chi phí nhất. Được bonus thêm năng lực xài các toán tử `>`, `<`, `=` và cả `ORDER BY` êm ái.
- **Cú pháp:**
  ```sql
  -- Dấu ->> là móc cái email ra và biến nó thành kiểu TEXT string tự nhiên
  CREATE INDEX idx_users_email ON Users ((attributes->>'email'));
  ```
- **Query ăn Index:**

```
  SELECT * FROM Users WHERE attributes->>'email' = 'john@example.com';
```

**Tóm tắt "Bùa hộ mệnh" trả lời phỏng vấn:**

> _"Để truy vấn JSONB không bị quét toàn bảng, ta có 3 chiến thuật dắt lưng: **Thứ nhất**, nếu cần soi xét linh hoạt không biết trước key, hãy dùng **GIN Index mặc định** để xài nhiều toán tử. **Thứ hai**, nếu biết chắc chỉ xài toán tử chứa `@>` và muốn tiết kiệm không gian đĩa nặng nề, hãy dùng GIN kèm cờ **jsonb_path_ops**. **Cuối cùng**, nếu chỉ thường xuyên chọc vào đúng 1 trường cố định trong JSON để tìm đích danh hoặc sắp xếp, hãy dùng `->>` móc nó ra thành đoạn Text và đánh **B-Tree Index** truyền thống, đây là phương án tối giản và nhẹ nhàng nhất cho hệ thống."_

---

### 13. Câu hỏi thực tế (Hệ thống lớn): Cần JOIN 2 bảng, mỗi bảng đều lấy ra 100.000 bản ghi, nội dung cả 2 bảng đều chứa 1 cột `JSONB` rất lớn (chứa cả 1 trang web). Cách xử lý để DB không sập?

Đây là một câu hỏi phân loại ứng viên lên thẳng Level **Senior/Technical Lead**. Bài toán này đánh thẳng vào việc Hệ quản trị CSDL xử lý một lượng I/O Disk và RAM khổng lồ khi tải nội dung văn bản (Text/JSON nặng) trong bộ nhớ để làm toán (JOIN).

Để trả lời "hạ gục" người đối diện, bạn cần trình bày theo 3 lớp: Hiểu cơ chế vật lý (TOAST), Tối ưu SQL, và Đổi kiến trúc lưu trữ.

#### Lớp 1: Bắt bệnh bằng cơ chế TOAST của PostgreSQL (hoặc LOB ở SQL Server)

- Khi bạn lưu một cục dữ liệu `JSONB` hoặc `TEXT` khổng lồ (như mã HTML của 1 trang web), PostgreSQL **không bao giờ lưu cục thịt đó ở bảng chính** (Main heap). Khung của 1 Data Page là 8KB, vượt quá nó sẽ xé nhỏ cục dữ liệu đó, nén lại và mang ra một bảng ẩn khác gọi là **bảng TOAST** (The Oversized-Attribute Storage Technique). Ở bảng chính chỉ giữ lại một "Con trỏ" (Pointer) trạc cỡ vài byte trỏ ra bảng TOAST.
- **Rủi ro chí mạng:** Khi lệnh SQL quét 100.000 dòng để đi `JOIN` mà bạn lỡ tay có chữ `SELECT *` hoặc `SELECT a.json_bự, b.json_bự`, thì CSDL phải dùng 100,000 cái con trỏ hộc tốc phi ra bảng TOAST ngoại vi, đọc đĩa (I/O) để lôi dữ liệu lên, giải nén (Decompress) cục HTML đó và nhồi vào RAM. Database sẽ lập tức ăn cạn kiệt Memory (OOM - Out of memory) khiến Server treo giò hoặc Time out hoàn toàn.

#### Lớp 2: Xử lý tầng Database (Tinh chỉnh luồng SQL bằng Kỹ thuật Deferred Join)

Luật bất thành văn: **Tuyệt đối không được kéo cục `JSONB/TEXT` lên RAM trong quá trình đang tính toán logic JOIN/Filter/Sort.**

Hãy áp dụng **Deferred Join (JOIN trì hoãn)**:

- **Bước 1:** Viết một Subquery chỉ thực hiện móc nối (JOIN) trên các cột nhỏ bé (như ID, status, user_id...) để lọc ra chính xác tập kết quả cuối cùng. Lúc này bộ nhớ RAM hoạt động cực kỳ mượt mã vì chỉ chứa toàn các con số Int nhẹ tênh.
- **Bước 2:** Xử lý phân trang (`LIMIT` / `OFFSET`) ngay tại Subquery đó để bóp nát từ 100.000 dòng kết quả (đã khớp) xuống chỉ còn ví dụ 20 dòng thực tế cần hiển thị trên UI cho User.
- **Bước 3:** Bốc đúng 20 cái ID kết quả vừa thắt cổ chai xong đi `JOIN ngược lại` với chính 2 bảng gốc để lần đầu tiên "móc" 2 cột `JSONB` TOAST khổng lồ ra. Lúc này CSDL chỉ phải lặn xuống ổ cứng đọc TOAST đúng **20 lần** thay vì 100,000 lần.

**Ví dụ Code SQL ghi điểm tuyệt đối:**

```sql
SELECT A.id, A.json_khong_lo, B.id, B.json_sieu_to
FROM (
    -- Subquery này siêu nhẹ: Phép màu của các con số
    SELECT t1.id AS table_a_id, t2.id AS table_b_id
    FROM TableA t1
    JOIN TableB t2 ON t1.ref_id = t2.id
    WHERE t1.status = 'ACTIVE'
    ORDER BY t1.created_at DESC
    LIMIT 20 OFFSET 0
) AS FastIDs
-- Bây giờ mới dùng 20 cái ID lôi phần xác JSON ra đưa cho Client
JOIN TableA A ON FastIDs.table_a_id = A.id
JOIN TableB B ON FastIDs.table_b_id = B.id;
```

#### Lớp 3: Xử lý tầng Kiến trúc System Design (Bức tường cuối cùng)

Nếu hệ thống bắt buộc phải **Xuất file Excel / Phân tích nội dung** của toàn bộ 100.000 trang web đó cùng lúc (Không thể dùng `LIMIT 20`), thì SQL Database đã hết giới hạn. Phải áp dụng thiết kế lại hệ thống (Architecure Migration):

- **Chuyển kho lưu trữ (Offloading Data Caching):** RDBMS không sinh ra để chứa cấu trúc file/HTML khổng lồ. Hãy chuyển nội dung code trang Web đó sang lưu ở **Object Storage (như Amazon S3)** hoặc **NoSQL phân tán (MongoDB / DynamoDB)**. Trong CSDL PostgreSQL, bảng của bạn chỉ làm nhiệm vụ lưu cột `s3_url_path`. Khi JOIN trả ra 100k kết quả đường link siêu tốc, việc tải nội dung khổng lồ 100k file HTML đó được đẩy cho Frontend/Client (chia nhỏ luồng network) gửi HTTP Request tự kéo thẳng về từ AWS chứ không được đè DB gánh.
- **Dùng Elasticsearch:** Nếu mục đích quét 100.000 dòng bằng code là để _Search text nội dung bên trong trang web (bóc HTML)_, thì quy trình này phải được đồng bộ (Sync) dữ liệu bảng vào **Elasticsearch**. ES dùng Inverted Index được thiết kế chuyên biệt để phân tích Text Search siêu quy mô và Distributed cực nhanh.

#### Lớp 4: Bài toán Case Study siêu khó (1 Triệu dòng AI Response & Citation)

**Tình huống:** Bạn có bảng `ai_responses` chứa cột `description` (Lưu chữ rất dài của AI) và bảng `citations` (lưu info website AI phân tích). Khi cần JOIN 2 bảng này để lọc ra dữ liệu của 1 thương hiệu (Brand) có tới **1 TRIỆU bản ghi**, dù có làm cách nào (kể cả Deferred Join) thì hệ thống cũng quay đều và Time Out nổ tung.

**Giải pháp hạ nốc ao nhà tuyển dụng:** _"Khi toán hạng lên tới 1 triệu hàng chứa Content bự, việc ép Relational Database (PostgreSQL/MySQL) thực hiện phép móc nối (JOIN) là một thiết kế sai lầm ngay từ đầu (Anti-pattern). Dù anh có Index tối thượng thì Network I/O và RAM múa con trỏ cũng sẽ sập. Em sẽ đập đi xây lại kiến trúc Query cho Data khổng lồ này."_

1.  **Chiến thuật 1: Phân mảnh dọc ngay từ lúc thiết kế DB (Vertical Partitioning - Tách bảng lõi)**
    Đây là kỹ năng Schema Design tuyệt đỉnh phòng tránh thảm họa: Cột văn bản siêu dài (`description` hoặc HTML/JSON) **không bao giờ nên gộp chung vào Bảng chính** (Core Table). Xé nó ra thành 1 bảng phụ (Ví dụ: `ai_responses_content` chỉ chừa 2 cột `response_id, description`). Bảng chính `ai_responses` chỉ giữ thân hình thon gọn bằng các cột ID, trạng thái, ngày tháng phục vụ Index.
    Lúc tìm file/lọc 1 triệu dòng thì JOIN trên bảng chính gọn nhẹ vô cùng. Chừng nào User thực sự muốn kéo chuột click vào xem chi tiết bài AI nào, ta mới query đúng 1 dòng qua bảng Phụ (Lazy Load) móc phần ruột chữ siêu bự ra màn hình. Đảm bảo tốc độ I/O đĩa cứng không bao giờ bị nghẽn mạng!

2.  **Chiến thuật 2: CQRS & ElasticSearch (Săn lùng văn bản)**
    Đồng bộ dữ liệu (Sync) của 2 bảng `ai_responses` và `citations` gộp chung lại thành 1 Document phẳng (Denormalized) và nhét vào **ElasticSearch (ES)**. ES sinh ra với cấu trúc Inverted Index, nó sinh ra để nuốt chửng hàng chục triệu hàng Text dài ngoằng. Thay vì gõ SQL JOIN 1 triệu hàng, ta gọi API sang ES filter theo `brand_id`. Tốc độ sẽ từ 30 giây tụt xuống còn 30 milliseconds.
3.  **Chiến thuật 3: Materialized View (Nấu chín dữ liệu trước - Pre-computation)**
    Nếu bài toán chỉ là để phục vụ màn hình **Dashboard Thống kê (Analytics)** hoặc báo cáo cuối ngày cho 1 Brand, không cần Real-time 100%. Ta sẽ không bắt DB phải JOIN 1 triệu dòng _ngay lúc User bấm nút_.
    Vào lúc 2h sáng (Lúc ít ai xài), ta chạy một Background Job dùng lệnh `CREATE MATERIALIZED VIEW` kết hợp JOIN 1 triệu dòng đó sẵn, tính toán tổng hợp rồi đúc ra thành 1 bảng kết quả vật lý mới toanh siêu nhẹ. Ban ngày User vào xem Brand đó thì chỉ việc `SELECT *` từ bảng kết tinh này, siêu nhanh vì cơm đã nấu sẵn từ đêm.
4.  **Chiến thuật 4: Pagination Rắn (Bẻ gãy tư duy User)**
    Hỏi ngược lại bài toán Business: _"Liệu có User/Hệ thống nào cần đọc 1 TRIỆU nội dung AI dài ngoằn ngoèo CÙNG 1 LÚC hay không?"_. Trình duyệt kéo 1 triệu dòng Node HTML DOM sẽ treo RAM máy tính người dùng trước cả khi Database sập.
    => Bắt buộc phá vỡ API thành **Cursor Pagination** (Chỉ lấy tối đa 50 dòng/lần). Có 1 triệu dòng thì kệ 1 triệu dòng, Database chỉ đọc Index B-Tree đi tìm 50 cái ID thỏa mãn `Brand_ID = X` rồi xách đi JOIN (Deferred Join). Câu lệnh sẽ lại nhẹ tựa lông hồng.
    Còn nếu mục đích lấy 1 triệu dòng là để máy tính xuất file Excel tải về (Export), ta phải bắn Query đó vào luồng **Message Queue (RabbitMQ/Kafka)** chạy ngầm (Asynchronous Worker). Khi nào Server âm thầm cày 1 triệu dòng JOIN gộp file CSV xong thì vẩy Notfication báo rinh file về.

**Tóm tắt "Bùa hộ mệnh" cho câu hỏi System Design:**

> _"Bài toán JOIN cột JSONB/Text dài sẽ làm sập RAM và Disk IO. Ngay từ khâu thiết kế, ta phải dùng **Vertical Partitioning** xé phần ruột chữ bự ra 1 bảng riêng biệt (Lazy Load) để bảng Chính giữ dáng thon gọn cho việc Index/Filter. Còn nến truy vấn bốc dữ liệu lên tới **1 Triệu dòng**, RDBMS đã cạn kiệt giới hạn vật lý. Ta bắt buộc chuyển kiến trúc: (1) Nhồi dữ liệu Denormalize sang **Elasticsearch** chuyên trị văn bản. (2) Đúc dữ liệu tính sẵn vào đêm qua bằng **Materialized Views**. (3) Dùng **Message Queue** ném việc xuất Excel 1 triệu dòng ra Worker chạy ngầm giải phóng API."_

---

### 14. Khi nào thì dùng Subquery (Truy vấn con)? Giải thích và Ví dụ thực tế

**Subquery (Truy vấn lồng / Truy vấn con)** là sức mạnh giúp bạn nhét nguyên một câu lệnh `SELECT` nhỏ vào bên trong một câu lệnh `SELECT`, `UPDATE`, hoặc `DELETE` lớn hơn.

#### Trả lời câu hỏi: "Khi nào thì nên dùng Subquery?"

Nhìn chung, bạn sẽ dùng Subquery trong 3 tình huống bắt buộc sau đây:

1. **Khi cần Lọc Dữ liệu dựa trên một Kết quả Động (Chưa biết trước):** Bạn muốn tìm "Ai là người lương cao nhất?", nhưng bạn méo biết số lương cao nhất là bao nhiêu. Bạn phải tạo 1 Subquery đi tìm con số MAX đó trước, rồi mới lấy nó làm thước đo cho câu Quary ngoài cùng.
2. **Khi cần Tạo ra một Bảng Ảo (Derived Table) để chạy tính toán tiếp:** Như tuyệt chiêu _Deferred Join_ bạn đã học ở trên, ta dùng Subquery gọt giũa 100k dòng xuống còn 20 ID trước, coi 20 ID đó như 1 "Bảng mới" rôi mới đem đi JOIN tiếp.
3. **Khi cần bóc tách từng Thuộc tính tính toán (Calculated Column):** Bạn muốn Select ra Tên nhân viên, kèm theo 1 cột đếm xem nhân viên đó đã chốt được bao nhiêu đơn hàng (SELECT lồng SELECT).

---

#### Cấp độ 1: Non-Correlated Subquery (Truy vấn con Độc Lập)

- **Đặc điểm:** Câu truy vấn con (Subquery) có thể **tự chạy một mình** mượt mà không cần mượn dữ liệu từ Câu truy vấn cha (Outer query).
- **Cơ chế:** Database sẽ thi hành Câu Subquery **TRƯỚC TIÊN** và chạy ĐÚNG 1 LẦN DUY NHẤT. Nó lấy kết quả đó làm hằng số đắp vào cho Câu lệnh Cha chạy sau cùng.

**Ví dụ thực tế:** _"Hãy tìm tất cả các nhân viên có mức lương cao hơn mức Mức lương Trung bình của toàn cty"_

```sql
-- Bước 1: Subquery (Tự chạy mượt mà ra con số giả sử là 15_000_000 đ)
-- SELECT AVG(salary) FROM Employees;

-- Bước 2: Ráp vào Câu Cha
SELECT id, name, salary
FROM Employees
WHERE salary > (SELECT AVG(salary) FROM Employees);
```

---

#### Cấp độ 2: Correlated Subquery (Truy vấn con Tương Quan / Phụ Thuộc)

Đây là sát thủ diệt Performance nếu dùng sai cách.

- **Đặc điểm:** Câu Subquery **KHÔNG THỂ tự chạy một mình** được. Nó phải mượn (tham chiếu) một Cột dữ liệu từ Câu Cha đưa vào thì nó mới chạy được.
- **Cơ chế:** Cứ **MỖI MỘT DÒNG** dữ liệu mà Câu Cha quét qua, nó lại phải ném tham số vào và KÍCH HOẠT Câu Subquery chạy lại từ đầu 1 lần. (Y hệt vòng lặp `for` lồng `for`).

**Ví dụ thực tế đắt giá:** _"Hãy in ra Tên của mỗi Phòng ban, VÀ KÈM THEO số lượng nhân viên đang làm ở phòng ban đó."_

**(Cách 1: Lỗi thời & Gây kẹt xe - Dùng Correlated Subquery ở vế SELECT)**

```sql
SELECT
    d.department_name,
    (
        -- Subquery này bị PHỤ THUỘC vào d.id của câu Cha bên ngoài
        -- Cứ mỗi vòng lặp 1 dòng Department, query này bị kích hoạt đi đếm lại 1 lần
        SELECT COUNT(*)
        FROM Employees e
        WHERE e.department_id = d.id
    ) AS total_employees
FROM Departments d;
```

_Nhược điểm:_ Nếu bảng `Departments` có 1.000 phòng ban, thì cái Subquery `SELECT COUNT` bên trong sẽ bị kích hoạt chạy tới 1.000 lần. Database tắc thở!

**(Cách 2: Tư duy chuẩn Senior - Ép 1000 lần Subquery đó thành 1 lần cấu trúc JOIN)**
Thay vì để Subquery nằm vắt vẻo trên `SELECT` để bị lặp lại, hãy đúc nó thành một **Bảng Ảo (Derived Table) nằm ở vế `FROM`** chỉ chạy đúng 1 lần:

```sql
SELECT
    d.department_name,
    emp_stats.total_employees
FROM Departments d
LEFT JOIN (
    -- Gộp nhóm (GROUP BY) đếm nhân viên cho TOÀN BỘ các phòng ban CÙNG 1 LÚC
    -- Đúc nó thành 1 cái bảng ảo cầm sẵn mang tên 'emp_stats'
    SELECT department_id, COUNT(*) AS total_employees
    FROM Employees
    GROUP BY department_id
) emp_stats ON d.id = emp_stats.department_id;
```

_Tiến hóa:_ Ở cách 2, Query Cha chạy 1 lần, Query Con (Bảng đếm) chạy 1 lần. Database nhẹ bẫng!

**Tóm tắt "Bùa hộ mệnh" đi Phỏng vấn:**

**Tóm tắt "Bùa hộ mệnh" đi Phỏng vấn:**

> _"Subquery nên được sử dụng khi ta cần kết quả động từ 1 truy vấn khác để làm tham số Lọc dữ liệu, hoặc để đúc ra một Bảng Ảo xử lý tính toán trung gian. Về Performance, hãy tranh thủ dùng **Non-Correlated Subquery** (truy vấn độc lập chạy 1 lần). Tuyệt đối cảnh giác với **Correlated Subquery** (truy vấn phụ thuộc vào cha) đặc biệt là đặt nó ở mệnh đề `SELECT` hay `WHERE` vì nó sẽ gây ra vòng lặp vô tận N+1 query. Nếu gặp trường hợp này, em sẽ Tối ưu lại bằng cách nhốt Subquery xuống mệnh đề `FROM` tạo thành Bảng Ảo và dùng `JOIN` để gom nhóm 1 lần duy nhất."_

---

### 15. Database Transaction là gì? Các thành phần ảo diệu (ACID) và Cách hoạt động biên chế

Câu hỏi về **Transaction (Giao dịch)** là câu hỏi "sinh tử" để đánh giá xem ứng viên có đủ trình độ để đụng vào logic liên quan đến **Tiền bạc, Thanh toán, và Dữ liệu nhạy cảm** hay không.

#### 1. Transaction (Giao dịch) là gì?

Transaction là một **"Gói công việc"** bao gồm nhiều thao tác (nhiều lệnh Insert/Update/Delete) được DataBase xử lý như thể chúng là **MỘT KHỐI THỐNG NHẤT DUY NHẤT**.

- Nếu tất cả các lệnh trong khối trót lọt -> Cả khối được lưu lại vĩnh viễn (`COMMIT`).
- Nếu chỉ 1 lệnh bị lỗi/đứt mạng giữa chừng -> Cả khối tự động hủy bỏ, quay tay đưa dữ liệu về y hệt trạng thái ban đầu (`ROLLBACK`).

**Ví dụ kinh điển:** Bài toán chuyển 100k từ tài khoản Alice sang Bob.

- Lệnh 1: Trừ 100k của Alice (UPDATE Alice_bal = bal - 100k)
- Lệnh 2: Cộng 100k cho Bob (UPDATE Bob_bal = bal + 100k)
  _Thảm họa:_ Lệnh 1 xong, tự dưng server cúp điện. Alice mất 100k mà Bob chưa nhận được!
  =>> Bọc 2 lệnh trên vào trong chữ `BEGIN TRANSACTION;` và `COMMIT;`. Nếu cúp điện giữa chừng, mở máy lên tiền Alice tự hoàn về y cũ.

---

#### 2. Các thành phần tạo nên sức mạnh (Nguyên lý ACID)

Bất kỳ CSDL Quan hệ (Relational DB) nào cũng phải tuân thủ nghiêm ngặt 4 nguyên tắc Vàng (Viết tắt là **A.C.I.D**):

1. **A - Atomicity (Tính Nguyên tử): "All or Nothing - Trọn vẹn hoặc Không gì cả"**
   - Đảm bảo toàn bộ gói lệnh hoàn thành 100%, hoặc thất bại toàn tập. Không có chuyện dữ liệu được ghi "nửa vời" vào DB.
2. **C - Consistency (Tính Nhất quán): "Luật là Luật"**
   - Từ lúc mở Transaction đến lúc Commit xong, dữ liệu trước và sau đó đều TRƯỚC SAU NHƯ MỘT tuân thủ đúng mọi ràng buộc Data (Foreign keys, Unique, Not Null, Check Constraint). Nếu sau khi trừ tiền mà tài khoản Alice bị âm (vi phạm Check Constraint >= 0) thì lập tức bắt Rollback toàn bộ.
3. **I - Isolation (Tính Cô lập): "Việc ai nấy làm"**
   - Sự diệu kỳ khi có 100 người cùng xúm vào giành mua 1 món hàng. Tính cô lập đảm bảo các Transaction chạy // (song song) không được phép "giẫm đạp", nhìn trộm (Dirty Read) hay phá hỏng dữ liệu của nhau khi chúng đang chưa chạy xong (Chưa Commit). DB dùng cơ chế **Lock (Khóa)** để xếp hàng bắt tụi nó đợi nhau.
4. **D - Durability (Tính Bền bỉ): "Chết cứng dính ngắc"**
   - Một khi chữ `COMMIT` đã được DB gật đầu "Thành Công!", thì đoạn dữ liệu đó phải **SỐNG MÃI MÃI** khắc ghi vào Ổ cứng vật lý. Kể cả giây tiếp theo server bốc cháy nổ tung, thay ổ cứng mới cắm vào chạy thì dữ liệu lúc nãy vẫn còn chễm chệ ở đó.

---

#### 3. [Deep Dive] Cách Transaction thực hành phép màu bên dưới (Dành cho Senior)

Làm thế nào Database đảm bảo được tính **Atomicity (Hoàn tác khi lỗi)** và tính **Durability (Bền bỉ khi cháy nổ)**? Nó diễn ra thế này:

1. **Cơ chế ghi sổ WAL (Write-Ahead Logging):**
   - RDBMS (như PostgreSQL) không bao giờ dại dột đem lệnh UPDATE của bạn nghiến thẳng xuống cái ổ cứng chậm chạp ngay lập tức.
   - Đầu tiên, nó viết "Toàn bộ ý định" (Dòng này bị đổi số 1 thành số 2 nà) vào một file Nhật ký Siêu tốc (Log File - hay còn gọi là WAL). Cứ ghi vào Nhật ký là tính là `XONG` rồi (Đảm bảo Bền bỉ `D`). Dù sập nguồn ngay lúc đó, khi khởi động lại, DB đọc lướt lại cuốn nhật ký để "Phục dựng" (Redo/Undo log) lại toàn bộ tiến độ đang dang dở.
2. **Cơ chế khóa Lock (MVCC - Multi-Version Concurrency Control):**
   - Để đảm bảo tính Cô lập (`I`), thay vì nhốt cứng (khóa nguyên cái bảng) bắt người khác phải đợi khi 1 user đang sửa data.
   - DB nhạy bén sẽ dùng MVCC: Nó âm thầm "chụp ảnh kỹ thuật số rẹt rẹt" (Snapshot) nhân bản dòng dữ liệu đó lên làm nhiều phiên bản tạm thời trong RAM. Người đang sửa cứ sửa trên bản Version 2, người đang đọc (`SELECT`) cứ đọc bình yên ở bản cũ Version 1 không bị vấp ngã văng lỗi chờ đợi. Khi người kia `COMMIT` xong, Version 1 tự hủy, Version 2 thành chánh cung.

**Tóm tắt "Bùa hộ mệnh" đi Phỏng vấn:**

> _"Transaction là một gói gom cụm nhiều lệnh Query lại với nhau để xử lý duy nhất 1 lần (All or Nothing) nhằm bảo vệ dữ liệu cực kỳ nhạy cảm. Nó phải tuân thủ 4 tính chất sống còn là **A.C.I.D**: **Atomicity** (Nguyên tử - Trọn vẹn hoặc dẹp hết), **Consistency** (Nhất quán - phải đúng mọi ràng buộc trước sau), **Isolation** (Cô lập - các giao dịch song song bị khóa tách biệt không dẫm đạp nhau), và **Durability** (Bền bỉ - Đã Commit là ghim vĩnh viễn vào đĩa cứng). Ở tầng dưới, Hệ quản trị gánh được 4 phép màu này chủ yếu nhờ vào file nhật ký **WAL (Write-Ahead Log)** giúp phục hồi dữ liệu khi cúp điện và bộ khóa đa phiên bản **MVCC** giúp đọc ghi song song mạnh mẽ."_

---

### 16. Làm cách nào đổi giá trị cột từ số thành boolean?

Có hai việc thường gặp: (1) **chuẩn hóa dữ liệu** — đổi giá trị số (0/1, 1/2…) sang true/false; (2) **đổi kiểu cột** — chuyển cột từ kiểu số (INT, TINYINT, SMALLINT…) sang kiểu boolean/bit của từng DB.

**Quy trình chuẩn (hai bước):**

1. **Chuẩn hóa về 0 và 1:** Đảm bảo trong cột chỉ còn hai giá trị: **0** và **1** (mọi số khác 0, ví dụ 2, -1, 99… đều map về **1**). Bước này dùng `UPDATE ... CASE WHEN` (hoặc tương đương) để dữ liệu “sạch” trước khi đổi kiểu hoặc dùng làm boolean.
2. **Setup quy ước:** **1 = true**, **0 = false**. DB (BOOLEAN/BIT/TINYINT(1)) và ứng dụng (ORM, driver) đều hiểu theo quy ước này; không cần lưu chữ "true"/"false" trong DB, chỉ cần 0/1 là đủ.

Sau khi chuẩn hóa xong, có thể giữ nguyên kiểu số (chỉ dùng 0/1) hoặc đổi kiểu cột sang boolean/bit tùy từng DB.

#### 1. Chỉ cập nhật giá trị (giữ nguyên kiểu cột)

Nếu cột vẫn là số nhưng bạn muốn chuẩn hóa về 0 = false, 1 = true (hoặc ngược lại), dùng `UPDATE` với điều kiện:

```sql
-- Quy ước: 0 → false (0), khác 0 → true (1)
UPDATE my_table
SET my_column = CASE
    WHEN my_column = 0 THEN 0
    ELSE 1
END
WHERE ...;  -- nên có WHERE để tránh sửa toàn bảng
```

Hoặc nếu DB hỗ trợ kiểu boolean và cột đã là boolean, có thể dùng trực tiếp:

```sql
UPDATE my_table SET is_active = (status = 1);
```

#### 2. Đổi kiểu cột từ số sang boolean (hoặc bit)

**PostgreSQL (có kiểu BOOLEAN):**

- Dùng `ALTER COLUMN ... TYPE ... USING` để vừa đổi kiểu vừa ánh xạ giá trị cũ (vd. 0 → false, khác 0 → true):

```sql
ALTER TABLE my_table
ALTER COLUMN my_column TYPE BOOLEAN
USING (my_column = 1);   -- hoặc USING (my_column::int <> 0) nếu cột là text
```

**MySQL / MariaDB:**

- Không có kiểu BOOLEAN riêng; `BOOLEAN` là alias của `TINYINT(1)`. Giá trị 0/1 vẫn là số. Có thể:
  - Giữ cột `TINYINT(1)` và trong ứng dụng coi 0 = false, 1 = true; hoặc
  - Đổi tên/comment cột cho rõ nghĩa, không bắt buộc đổi kiểu.

Nếu muốn “đổi kiểu” cho đồng bộ (vd. từ INT sang TINYINT(1)):

```sql
ALTER TABLE my_table
MODIFY COLUMN my_column TINYINT(1) NOT NULL DEFAULT 0;
-- Sau đó có thể chuẩn hóa giá trị bằng UPDATE như trên
```

**SQL Server (dùng BIT):**

- Kiểu boolean thực tế là `BIT` (0/1):

```sql
-- Đổi cột từ INT (hoặc số) sang BIT, ánh xạ 0 → 0, khác 0 → 1
ALTER TABLE my_table
ADD is_active_new BIT NULL;

UPDATE my_table SET is_active_new = CASE WHEN my_column = 0 THEN 0 ELSE 1 END;

ALTER TABLE my_table DROP COLUMN my_column;
EXEC sp_rename 'my_table.is_active_new', 'my_column', 'COLUMN';
```

Hoặc nếu cho phép thêm cột rồi xóa cột cũ, có thể dùng một lần `ALTER` với computed conversion (tùy phiên bản).

#### 3. Lưu ý khi đổi kiểu

- **Ràng buộc và default:** Sau khi đổi kiểu, kiểm tra lại `NOT NULL`, `DEFAULT`, `CHECK` (vd. chỉ cho phép 0/1 hoặc true/false).
- **Index:** Index trên cột số vẫn dùng được sau khi chuyển sang boolean/bit; có thể rebuild index nếu DB yêu cầu hoặc để tối ưu.
- **Ứng dụng:** Đảm bảo driver/ORM map đúng kiểu boolean/bit (true/false hoặc 0/1) để không lệch với DB.

**Tóm tắt:**

> _Quy trình chuẩn: **(1) Chuẩn hóa về 0 và 1** — dùng `UPDATE ... SET col = CASE WHEN col = 0 THEN 0 ELSE 1 END` để mọi giá trị khác 0 thành 1; **(2) Setup quy ước 1 = true, 0 = false** — DB và ứng dụng dùng chung quy ước này. Sau đó có thể giữ kiểu số hoặc đổi kiểu cột: PostgreSQL dùng `ALTER COLUMN ... TYPE BOOLEAN USING (col = 1)`; MySQL giữ `TINYINT(1)`; SQL Server dùng `BIT`. Luôn chuẩn hóa (bước 1) trước hoặc trong lúc đổi kiểu để tránh lỗi._

---

### 17. Update số dư với TypeORM — tại sao bị “số cuối cùng” (ghi đè) thay vì cộng vào?

Khi dùng TypeORM (hoặc ORM nói chung) theo kiểu **đọc entity → sửa thuộc tính → save**, bạn đang ghi **một giá trị tuyệt đối** xuống DB. ORM sinh ra câu lệnh dạng `UPDATE ... SET balance = :value`, không phải `SET balance = balance + :delta`. Vì vậy nếu hai request cùng lúc “cộng” vào số dư, hoặc bạn tưởng là “cộng” nhưng lại set bằng số đã tính trên bản copy cũ, kết quả sẽ là **ghi đè** — chỉ số cuối cùng được ghi, mất cập nhật của người khác hoặc của chính request trước đó.

#### Ví dụ cụ thể (sai)

**Dữ liệu ban đầu:** `User` id=1, `balance = 100`.

**Request A (nạp tiền +50):**

```typescript
const user = await userRepo.findOne({ where: { id: 1 } }); // balance = 100
user.balance = user.balance + 50;                          // 150 (trong memory)
await userRepo.save(user);                                // UPDATE users SET balance = 150 WHERE id = 1
```

**Request B (nạp tiền +30) chạy gần cùng lúc:**

```typescript
const user = await userRepo.findOne({ where: { id: 1 } }); // vẫn đọc được 100 (chưa commit A)
user.balance = user.balance + 30;                          // 130 (trong memory)
await userRepo.save(user);                                // UPDATE users SET balance = 130 WHERE id = 1
```

Kết quả: **balance = 130**. Request A (150) bị request B ghi đè. Đúng ra phải là 100 + 50 + 30 = **180**.  
Nguyên nhân: mỗi request đều **đọc một giá trị cũ**, cộng trong app rồi **ghi đè** bằng một con số tuyệt đối; không có phép “cộng thêm” thực sự trong DB.

#### Cách xử lý đúng — cộng nguyên tử trong DB

Thay vì “đọc → cộng trong code → ghi giá trị tuyệt đối”, hãy để **database** thực hiện phép cộng **một lần** trong câu `UPDATE`: `SET balance = balance + :delta`. Chỉ cần truyền **số tiền thay đổi** (delta), không truyền số dư mới.

**Cách 1: QueryBuilder (nên dùng)**

```typescript
await dataSource
  .getRepository(User)
  .createQueryBuilder()
  .update(User)
  .set({ balance: () => 'balance + :delta' })
  .setParameter('delta', 50)
  .where('id = :id', { id: 1 })
  .execute();
```

Câu SQL sinh ra dạng: `UPDATE users SET balance = balance + 50 WHERE id = 1`. Mọi request đều “cộng thêm” vào giá trị hiện tại trong DB, không ghi đè.

**Cộng hoặc trừ (nạp / rút):**

```typescript
// Nạp +50
.set({ balance: () => 'balance + :delta' })
.setParameter('delta', 50);

// Rút 20 (cộng số âm)
.set({ balance: () => 'balance + :delta' })
.setParameter('delta', -20);
```

**Cách 2: Raw query (khi cần)**

```typescript
await dataSource.query(
  'UPDATE users SET balance = balance + $1 WHERE id = $2',
  [50, 1]
);
```

**Cách 3: Dùng transaction + lock (nếu bắt buộc phải đọc rồi mới quyết định cộng bao nhiêu)**

Nếu logic phức tạp (vd. kiểm tra hạn mức rồi mới cộng), có thể khóa dòng khi đọc rồi update trong cùng transaction:

```typescript
await dataSource.transaction(async (manager) => {
  const user = await manager.getRepository(User).findOne({
    where: { id: 1 },
    lock: { mode: 'pessimistic_write' }, // SELECT ... FOR UPDATE
  });
  if (!user) throw new Error('User not found');
  // Kiểm tra điều kiện, tính delta...
  await manager.getRepository(User).update(
    { id: 1 },
    { balance: user.balance + 50 }
  );
});
```

Ưu tiên vẫn là **cộng nguyên tử** (QueryBuilder hoặc raw `balance = balance + :delta`) để đơn giản và tránh lost update.

---

### 18. Ví dụ: Thiết kế DB hệ thống bán sách và các truy vấn phức tạp

Phần này dùng **hệ thống bán sách** làm ví dụ: thiết kế schema và các **truy vấn phức tạp** để demo.

**Thiết kế schema (tóm tắt):**

- **users** (id, email, full_name, phone, created_at)
- **addresses** (id, user_id FK, label, street, city, district, is_default)
- **categories** (id, name, slug, parent_id FK — cây thể loại)
- **authors** (id, name, bio)
- **books** (id, title, isbn, category_id FK, price, stock_quantity, published_at)
- **book_authors** (book_id FK, author_id FK — N–N)
- **orders** (id, user_id FK, address_id FK, status, total_amount, coupon_code, created_at)
- **order_items** (id, order_id FK, book_id FK, quantity, unit_price)
- **coupons** (code PK, discount_type, discount_value, min_order_amount, valid_from, valid_until)

**Các truy vấn khó (demo):**

1. **Top 5 khách mua nhiều tiền nhất 6 tháng:** JOIN users–orders, WHERE status + created_at, GROUP BY user, SUM(total_amount), ORDER BY DESC LIMIT 5.
2. **Top 10 sách bán chạy trong quý:** JOIN books–order_items–orders, SUM(quantity), GROUP BY book, lọc quý, LIMIT 10.
3. **Đơn có tổng tiền > trung bình đơn của chính user:** correlated subquery — WHERE total_amount > (SELECT AVG(total_amount) FROM orders WHERE user_id = o.user_id).
4. **Thể loại doanh thu cao nhất năm:** JOIN categories–books–order_items–orders, SUM(quantity*unit_price), GROUP BY category, lọc năm, ORDER BY revenue DESC LIMIT 1.
5. **User chưa từng mua sách thể loại "Tiểu thuyết":** NOT EXISTS (orders → order_items → books → categories WHERE category.name = 'Tiểu thuyết' AND order.user_id = users.id).
6. **Xếp hạng doanh thu sách trong từng thể loại:** CTE tính revenue theo book+category, RANK() OVER (PARTITION BY category_id ORDER BY revenue DESC).
7. **Sách bán trong tháng > trung bình bán trong tháng của thể loại:** 2 CTE — (1) monthly_book_sales, (2) category_avg; JOIN và WHERE qty_sold > avg_qty.
8. **Đơn có nhiều đầu sách (distinct book) nhất trong tháng:** CTE COUNT(DISTINCT book_id) theo order, WHERE distinct_books = (SELECT MAX(distinct_books) FROM CTE).

Khi demo: nhấn mạnh lọc status đơn (paid/shipping/delivered), lọc thời gian, index (orders.user_id+created_at, order_items.order_id, books.category_id).

**Schema SQL đầy đủ (PostgreSQL):**

```sql
CREATE TABLE users ( id SERIAL PRIMARY KEY, email VARCHAR(255) UNIQUE NOT NULL, full_name VARCHAR(255), phone VARCHAR(50), created_at TIMESTAMPTZ DEFAULT NOW() );
CREATE TABLE addresses ( id SERIAL PRIMARY KEY, user_id INT NOT NULL REFERENCES users(id) ON DELETE CASCADE, label VARCHAR(100), street TEXT, city VARCHAR(100), district VARCHAR(100), is_default BOOLEAN DEFAULT FALSE );
CREATE TABLE categories ( id SERIAL PRIMARY KEY, name VARCHAR(200) NOT NULL, slug VARCHAR(200) UNIQUE, parent_id INT REFERENCES categories(id) );
CREATE TABLE authors ( id SERIAL PRIMARY KEY, name VARCHAR(255) NOT NULL, bio TEXT );
CREATE TABLE books ( id SERIAL PRIMARY KEY, title VARCHAR(500) NOT NULL, isbn VARCHAR(20) UNIQUE, category_id INT REFERENCES categories(id), price DECIMAL(12,2) NOT NULL, stock_quantity INT NOT NULL DEFAULT 0, published_at DATE );
CREATE TABLE book_authors ( book_id INT REFERENCES books(id) ON DELETE CASCADE, author_id INT REFERENCES authors(id) ON DELETE CASCADE, PRIMARY KEY (book_id, author_id) );
CREATE TABLE coupons ( code VARCHAR(50) PRIMARY KEY, discount_type VARCHAR(10) NOT NULL, discount_value DECIMAL(12,2) NOT NULL, min_order_amount DECIMAL(14,2) DEFAULT 0, valid_from TIMESTAMPTZ, valid_until TIMESTAMPTZ );
CREATE TABLE orders ( id SERIAL PRIMARY KEY, user_id INT NOT NULL REFERENCES users(id), address_id INT REFERENCES addresses(id), status VARCHAR(20) NOT NULL DEFAULT 'draft', total_amount DECIMAL(14,2), coupon_code VARCHAR(50) REFERENCES coupons(code), created_at TIMESTAMPTZ DEFAULT NOW() );
CREATE TABLE order_items ( id SERIAL PRIMARY KEY, order_id INT NOT NULL REFERENCES orders(id) ON DELETE CASCADE, book_id INT NOT NULL REFERENCES books(id), quantity INT NOT NULL CHECK (quantity > 0), unit_price DECIMAL(12,2) NOT NULL );
CREATE INDEX idx_orders_user_created ON orders(user_id, created_at DESC);
CREATE INDEX idx_order_items_order ON order_items(order_id);
CREATE INDEX idx_books_category ON books(category_id);
```

**Ví dụ query đầy đủ (3 câu khó tiêu biểu):**

```sql
-- Query 3: Đơn có tổng tiền > trung bình đơn của chính user (correlated subquery)
SELECT o.id, o.user_id, o.total_amount, o.created_at
FROM orders o
WHERE o.status IN ('paid', 'shipping', 'delivered')
  AND o.total_amount > ( SELECT AVG(o2.total_amount) FROM orders o2 WHERE o2.user_id = o.user_id AND o2.status IN ('paid', 'shipping', 'delivered') )
ORDER BY o.user_id, o.total_amount DESC;

-- Query 5: User chưa từng mua sách thể loại "Tiểu thuyết" (NOT EXISTS)
SELECT u.id, u.full_name, u.email FROM users u
WHERE NOT EXISTS (
  SELECT 1 FROM orders o
  JOIN order_items oi ON oi.order_id = o.id
  JOIN books b ON b.id = oi.book_id
  JOIN categories c ON c.id = b.category_id
  WHERE o.user_id = u.id AND o.status IN ('paid', 'shipping', 'delivered') AND c.name = 'Tiểu thuyết'
);

-- Query 6: Xếp hạng doanh thu sách trong từng thể loại (RANK window)
WITH book_revenue AS (
  SELECT b.id AS book_id, b.title, b.category_id, SUM(oi.quantity * oi.unit_price) AS revenue
  FROM books b JOIN order_items oi ON oi.book_id = b.id JOIN orders o ON o.id = oi.order_id
  WHERE o.status IN ('paid', 'shipping', 'delivered') GROUP BY b.id, b.title, b.category_id
)
SELECT book_id, title, category_id, revenue, RANK() OVER (PARTITION BY category_id ORDER BY revenue DESC NULLS LAST) AS rank_in_category
FROM book_revenue ORDER BY category_id, rank_in_category;
```

#### Tóm tắt

- **Tại sao bị “số cuối cùng”:** TypeORM `save(entity)` luôn ghi **giá trị tuyệt đối** của thuộc tính (`UPDATE ... SET balance = :value`). Hai request cùng đọc 100, mỗi bên cộng rồi ghi 150 và 130 → ghi đè, mất một phần cập nhật.
- **Cách xử lý:** Không “đọc → cộng trong code → save”. Dùng **cộng trong DB**: QueryBuilder `.set({ balance: () => 'balance + :delta' }).setParameter('delta', 50)` hoặc raw `UPDATE ... SET balance = balance + $1`. Như vậy mọi thay đổi đều là “cộng thêm” vào giá trị hiện tại, không ghi đè.


---

### 19. N+1 query là gì? Cách khắc phục

**N+1 query** là vấn đề hiệu năng: thay vì lấy dữ liệu liên quan trong **một (hoặc ít) câu query**, ứng dụng chạy **1 query** lấy danh sách N bản ghi, rồi với **mỗi bản ghi** lại chạy **thêm 1 query** để lấy quan hệ → tổng cộng **1 + N** query. Khi N lớn (vd. 100 đơn hàng), DB bị đánh 101 lần, chậm và tốn tài nguyên.

#### Ví dụ gây N+1

**Ngữ cảnh:** Cần trả về danh sách đơn hàng và **các dòng đơn (order_items)** của từng đơn.

**Cách sai (N+1):**

1. Query 1: `SELECT * FROM orders WHERE user_id = 1` → trả về 50 đơn.
2. Với mỗi đơn, gọi thêm: `SELECT * FROM order_items WHERE order_id = ?` → 50 query nữa.
3. Tổng: **1 + 50 = 51 query**.

Code kiểu này thường xuất hiện khi dùng ORM **lazy loading**: lấy danh sách `orders`, rồi trong vòng lặp truy cập `order.items` (hoặc `order.getItems()`) → ORM tự chạy query mỗi lần.

```typescript
// Ví dụ TypeORM - dễ gây N+1
const orders = await orderRepo.find({ where: { user_id: 1 } });
for (const order of orders) {
  const items = await orderItemRepo.find({ where: { order_id: order.id } });
  order.items = items;
}
```

#### Cách khắc phục

**1. Eager load (load quan hệ cùng lúc)**

ORM cho phép load liên kết trong một lần gọi: dữ liệu quan hệ được lấy bằng JOIN (hoặc query in batch) do ORM sinh ra, không cần N query riêng.

```typescript
// TypeORM: relations
const orders = await orderRepo.find({
  where: { user_id: 1 },
  relations: ['items', 'items.book'],
});
// Thường chỉ 1–2 query (orders + items, hoặc JOIN)
```

**2. Một query JOIN (raw / QueryBuilder)**

Lấy luôn orders và order_items trong một câu SQL, nhóm (group) hoặc map trong code.

```sql
SELECT o.*, oi.id AS item_id, oi.book_id, oi.quantity, oi.unit_price
FROM orders o
LEFT JOIN order_items oi ON oi.order_id = o.id
WHERE o.user_id = 1;
```

Ứng dụng nhóm theo `o.id` để gắn items vào từng order. Chỉ **1 query**.

**3. Batch load (2 query: 1 lấy danh sách, 1 lấy tất cả quan hệ)**

- Query 1: `SELECT * FROM orders WHERE user_id = 1` → có danh sách `order_ids`.
- Query 2: `SELECT * FROM order_items WHERE order_id IN (1, 2, 3, ...)` → lấy toàn bộ items của các đơn đó.
- Trong code: map `order_id` → mảng items, gán vào từng order.

Tổng **2 query** bất kể N đơn.

```typescript
const orders = await orderRepo.find({ where: { user_id: 1 } });
const ids = orders.map((o) => o.id);
const allItems = await orderItemRepo.find({ where: { order_id: In(ids) } });
const byOrderId = groupBy(allItems, 'order_id');
orders.forEach((o) => (o.items = byOrderId[o.id] ?? []));
```

**4. DataLoader (GraphQL / batch theo request)**

Trong môi trường GraphQL (hoặc API trả về nested), mỗi field có thể gọi DB riêng → dễ N+1. **DataLoader** gom nhiều key (vd. nhiều `order_id`) trong cùng một "tick" thành một batch, gọi **một lần** `WHERE order_id IN (...)` rồi phân phối kết quả về từng key. Kết quả: mỗi loại quan hệ chỉ **1 query** cho cả request.

#### Tóm tắt

- **N+1:** 1 query lấy N bản ghi, rồi N query phụ để lấy quan hệ → **1 + N** query; rất chậm khi N lớn.
- **Nguyên nhân thường gặp:** Lazy loading trong ORM, vòng lặp gọi query theo từng bản ghi.
- **Khắc phục:** (1) **Eager load** (`relations` / include) để ORM lấy quan hệ trong ít query; (2) **Một query JOIN** rồi group trong code; (3) **Batch load:** 1 query lấy danh sách, 1 query `WHERE id IN (...)` lấy toàn bộ quan hệ rồi map; (4) **DataLoader** khi dùng GraphQL hoặc mô hình resolve từng field.
