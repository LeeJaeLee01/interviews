# Cấu trúc dữ liệu Tree (Cây)

## 1. Khái niệm cơ bản (Basic Concepts)

Tree (cây) là một cấu trúc dữ liệu phi tuyến tính (non-linear data structure) dùng để lưu trữ dữ liệu theo dạng phân cấp (hierarchical).
Cây bao gồm các nút (nodes) được kết nối với nhau bởi các cạnh (edges).

**Các thuật ngữ quan trọng:**

- **Root (Nút gốc):** Nút trên cùng của cây. Mỗi cây chỉ có một nút gốc.
- **Parent Node (Nút cha):** Nút có các nút con nối trực tiếp phía dưới nó.
- **Child Node (Nút con):** Nút nằm dưới một nút cha.
- **Leaf (Nút lá):** Nút không có bất kỳ nút con nào (nổi ở cuối cùng của các nhánh).
- **Sibling (Nút anh em):** Các nút có chung một nút cha.
- **Depth (Độ sâu của nút):** Khoảng cách (số lượng cạnh) từ nút gốc đến nút đó.
- **Height (Chiều cao của cây):** Số lượng cạnh lớn nhất từ nút gốc đến một nút lá xa nhất.

## 2. Các loại Tree phổ biến (Common Types of Trees)

1. **Binary Tree (Cây nhị phân):**
   - Mỗi nút có tối đa 2 nút con (con trái - left child và con phải - right child).

2. **Binary Search Tree - BST (Cây nhị phân tìm kiếm):**
   - Là cây nhị phân có tính chất: Mọi giá trị của nút con bên **trái** luôn **nhỏ hơn** giá trị nút cha, và mọi giá trị của nút con bên **phải** luôn **lớn hơn** giá trị nút cha.
   - Thao tác Search/Insert/Delete trung bình mất `O(log n)`. Tuy nhiên, trường hợp xấu nhất (cây bị lệch thành danh sách liên kết) sẽ tốn `O(n)`.

3. **Self-Balancing Trees (AVL Tree, Red-Black Tree):**
   - Là cây BST nhưng có khả năng tự quay và cân bằng lại sau các thao tác Insert/Delete để tránh bị lệch.
   - Đảm bảo độ phức tạp thời gian luôn luôn là `O(log n)` cho mọi thao tác.
   - _Ứng dụng:_ `Red-Black Tree` được dùng trong triển khai Map/Set của nhiều ngôn ngữ (C++, Java).

4. **B-Tree / B+ Tree:**
   - Cây tìm kiếm nhiều nhánh. Mỗi nút có thể chứa nhiều giá trị và có nhiều con.
   - Chiều cao cực kỳ thấp, giảm thiểu số lần đọc từ đĩa.
   - _Ứng dụng:_ Cấu trúc xương sống cho Indexing trong các Database (MySQL, PostgreSQL).

5. **Trie (Prefix Tree):**
   - Mỗi nút lưu một ký tự, đi từ gốc xuống lá tạo thành một từ.
   - Tìm kiếm cực kì nhanh gọn với chuỗi string (tốn `O(k)` với k là độ dài chuỗi).
   - _Ứng dụng:_ Autocomplete, Spell checker, IP routing (Longest prefix match).

6. **Heap (Min-Heap / Max-Heap):**
   - Là cây nhị phân hoàn chỉnh (Complete Binary Tree), luôn thỏa mãn tính chất: Nút cha luôn nhỏ hơn các nút con (Min-Heap) hoặc nút cha luôn lớn hơn các nút con (Max-Heap).
   - _Ứng dụng:_ Priority Queue (hàng đợi ưu tiên), thuật toán HeapSort.

## 3. Các cách duyệt cây (Tree Traversal)

Thay vì đi theo tuần tự từng index như mảng, với Tree chúng ta có các phương thức duyệt:

### Đi theo chiều sâu (Depth-First Search - DFS)

Thường được triển khai bằng đệ quy hoặc dùng Stack (ngăn xếp).

- **Pre-order (NLR - Nút, Trái, Phải):** Duyệt nút hiện tại, sau đó duyệt nhánh con trái, cuối cùng nhánh con phải. _(Thường dùng để copy cây)._
- **In-order (LNR - Trái, Nút, Phải):** Duyệt nhánh trái, nút hiện tại, nhánh phải. _(Nếu áp dụng trên BST, ta sẽ thu được danh sách các phần tử theo thứ tự tăng dần)._
- **Post-order (LRN - Trái, Phải, Nút):** Duyệt nhánh trái, nhánh phải, rồi mới duyệt nút hiện tại. _(Thường dùng khi muốn xoá cây, xoá các nút con trước rồi mới xoá gốc)._

### Đi theo chiều rộng (Breadth-First Search - BFS / Level-order)

- Quét từng tầng (level) của cây từ trên xuống dưới, từ trái sang phải.
- _(Thường làm bằng cách dùng cấu trúc dữ liệu Queue)._

## 4. Ưu/Nhược điểm và Tại sao sử dụng Tree?

**Tại sao dùng?**

1. **Phản ánh đúng dữ liệu:** Rất nhiều dữ liệu thực tế bản chất đã là phân cấp (Sơ đồ tổ chức công ty, File System trong máy tính, cấu trúc DOM HTML/XML).
2. **Hiệu suất (với BST/Balanced Tree):** Nhanh hơn Mảng (vì Array insert/delete ở giữa mất `O(n)`), và tìm kiếm nhanh hơn Linked List (tìm kiếm trong Linked list mất `O(n)`).
3. Sử dụng linh hoạt đặc thù theo cấu trúc (Trie tìm chuỗi rất vô đối, B-Tree giúp DBMS tối ưu tra cứu ổ cứng).

## 5. Cấy trúc mã cơ bản (Code minh hoạ) - Cây nhị phân và Pre-order

```python
class TreeNode:
    def __init__(self, value):
        self.value = value
        self.left = None
        self.right = None

def pre_order_traversal(root):
    if root is None:
        return
    print(root.value, end=" ")    # N
    pre_order_traversal(root.left)  # L
    pre_order_traversal(root.right) # R

# Tạo cây
#       1
#      / \
#     2   3
#    / \
#   4   5

root = TreeNode(1)
root.left = TreeNode(2)
root.right = TreeNode(3)
root.left.left = TreeNode(4)
root.left.right = TreeNode(5)

# Kết quả In ra: 1 2 4 5 3
```
