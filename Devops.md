# Hệ Thống & DevOps (Interview Prep)

---

### 1. Làm thế nào để Stop (Dừng) một Pod trong Kubernetes (K8s)?

Trong Kubernetes, Pod là thực thể phù du (ephemeral). Không có khái niệm "Stop/Pause rồi Start lại" y nguyên cái Pod đó như Docker Container. Bạn chỉ có thể **Xóa (Delete)** nó, và để ReplicaSet/Deployment tự động sinh ra một Pod mới thay thế (nếu có).

Tuy nhiên, có nhiều cấp độ để "dừng" hoặc "xóa" tùy theo bài toán:

#### Cách 1: Xóa Pod thông thường (Graceful Termination)

Đây là cách an toàn nhất. K8s sẽ gửi tín hiệu `SIGTERM` (mặc định cho 30 giây) để App bên trong có thời gian lưu data, đóng kết nối DB, hoàn thành nốt request đang chạy. Sau 30s nếu chưa chết, nó bồi thêm `SIGKILL`.

```bash
# Xóa 1 Pod cụ thể
kubectl delete pod <pod-name> -n <namespace>

# Xóa nhiều Pod cùng lúc theo Label
kubectl delete pod -l app=my-backend -n <namespace>
```

#### Cách 2: Xóa Ép buộc (Force Delete) - Khi Pod bị kẹt ở trạng thái Terminating

Nếu Node bị chết mạng hoặc App bị treo cứng không chịu tắt, Pod sẽ bị kẹt chữ `Terminating`. Lúc này ta phải mổ sống, không chờ 30 giây rườm rà nữa:

```bash
# Thêm cờ --force và --grace-period=0
kubectl delete pod <pod-name> -n <namespace> --force --grace-period=0
```

#### Cách 3: Muốn "Dừng hẳn" Pod mà không cho nó mọc lại (Scale down)

Nếu bạn dùng `kubectl delete pod` ở Cách 1, thì 3 giây sau Deployment controller sẽ tự động đẻ ra 1 cái Pod mới y chang vì nó cần duy trì số lượng Replicas. Nếu bạn THỰC SỰ muốn tắt hẳn con App đó nghỉ ngơi:

```bash
# Scale số lượng Pod của Deployment về con số 0
kubectl scale deployment <deployment-name> --replicas=0 -n <namespace>
```

#### Cách 4: Tắt Pod để Debug (Cô lập Pod)

Đôi khi bạn muốn con Pod "chết giả" so với người dùng nhưng thân xác nó vẫn còn để bạn chui vào bắt bug.
Bản chất K8s định tuyến Traffic vào Pod thông qua các **Label**. Bạn chỉ cần đổi Label của cái Pod đó là Route/Service sẽ lơ nó qua 1 bên (Ngưng nhận traffic).

```bash
# Xóa nhãn app=my-backend đi và thêm nhãn status=debug
kubectl label pod <pod-name> app- status=debug --overwrite
```

**Tóm tắt "Bùa hộ mệnh" đi Phỏng vấn:**

> _"Trong Kubernetes không có khái niệm Tạm dừng (Pause/Stop) Pod như Docker, mà chỉ có Xóa (Delete) đi tạo lại do tính chất phù du (ephemeral). Cách chuẩn mực là dùng `kubectl delete pod` để kích hoạt quá trình Graceful Termination (chờ 30s gửi SIGTERM đóng kết nối an toàn). Nếu Pod bị treo cứng Terminating, ta dùng cờ `--force --grace-period=0`. Còn nếu mục đích là vĩnh viễn tắt App không cho tự mọc lại, ta phải can thiệp vào tầng trên bằng lệnh `kubectl scale deployment --replicas=0`."_
