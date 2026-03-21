# Scale backend: Docker Compose vs Kubernetes (tóm tắt phỏng vấn)

> **Cách 1 (khuyến nghị khi muốn scale không sửa Compose mỗi lần):** Trong stack Docker có thêm **Nginx làm load balancer nội bộ** (một cổng publish ra host); các replica **`app`** chỉ nằm trong mạng Docker, scale bằng lệnh. **Nginx trên máy chủ** (HTTPS) chỉ cần trỏ vào **một** cổng đó.

---

## Cách 1 — Docker Compose + Nginx trong Docker (LB) + Nginx trên host

**Ý tưởng:** Tránh phải thêm `app2`, `app3` trong `docker-compose.yml` và sửa `upstream` trên OS mỗi lần tăng instance.

| Lớp | Vai trò |
|-----|--------|
| **`app` (scale N)** | Nhiều container cùng image, **không** `ports` ra host (chỉ trong `app-network`) |
| **`nginx-lb` (trong Compose)** | **Một** service publish ví dụ `3030:80` ra host; **`upstream`** tới `http://app:3030` — khi scale, DNS Docker gắn nhiều IP cho tên `app`, LB phân tải |
| Postgres / Redis | Một instance trong Compose, dùng chung |
| **Nginx trên máy chủ (HTTPS)** | `location /api/` → `proxy_pass http://127.0.0.1:3030/` (**một** điểm vào duy nhất vào LB Docker) |

**Scale (không sửa file Compose để thêm từng instance):**

```bash
docker compose up -d --scale app=3
```

**Ưu điểm:** Tăng/giảm số replica bằng lệnh; Nginx OS không cần thêm dòng `server` mỗi lần có thêm backend.

**Nhược điểm:** Phải có **một** container Nginx (hoặc HAProxy tương đương) trong Compose; cấu hình `resolver` + biến trong `proxy_pass` hoặc template cổng cho đúng `PORT` của app. Vẫn không có rolling update / HPA như K8s.

**So với mô hình “mỗi instance một cổng host”:** Cách cũ (chỉ Nginx OS, `127.0.0.1:3030`, `:3031`…) thường **phải sửa** Compose + file Nginx mỗi lần thêm instance — **không** dùng làm “cách 1” chính nếu mục tiêu là scale không đụng file.

---

## Cách 2 — Kubernetes (K8s)

**Ý tưởng:** Khai báo **Deployment** (replicas) + **Service** (ClusterIP) + **Ingress** (HTTPS, domain).

| Thành phần | Vai trò |
|------------|--------|
| `Deployment` + `Pod` | N replica của app, rolling update, health |
| `Service` (ClusterIP) | Load balance nội bộ cluster tới các Pod |
| `Ingress` + controller | HTTPS, host, path `/api` → Service |
| `Secret` / `ConfigMap` | Env, DB URL, Redis |

**Scale:**

```bash
kubectl scale deployment api --replicas=3
# hoặc HPA: tự scale theo CPU/RAM
```

**Ưu điểm:** Replica tự mô tả; rolling update; readiness/liveness; HPA; multi-node; phù hợp production lớn / nhiều team.

**Nhược điểm:** Học curve và vận hành cao hơn; cluster có chi phí (managed EKS/GKE/AKS hoặc tự vận hành).

---

## So sánh nhanh (nói trong interview)

| Tiêu chí | Docker Compose + **Nginx LB trong Docker** + Nginx OS | Kubernetes |
|----------|--------------------------------------------------------|------------|
| Scale lệnh | `docker compose up --scale app=N` (không thêm service `app2`…) | `kubectl scale` / HPA |
| Sửa file khi tăng replica? | **Không** (nếu đã thiết kế 1 LB + scale `app`) | **Không** (chỉ đổi `replicas`) |
| HTTPS / public | Nginx OS → một cổng LB Docker; hoặc TLS ở Ingress | Ingress + Service |

---

## Một câu kết (elevator pitch)

> *“Để scale ngang mà không sửa compose mỗi lần, em đặt **Nginx trong Docker** làm load balancer: các container `app` không publish port, chỉ scale `app`, còn **một** cổng public map ra LB. Nginx trên **máy chủ** chỉ `proxy_pass` vào đúng cổng đó. Khi cần rolling deploy, HA nhiều node, HPA thì chuyển lên **Kubernetes**: Deployment + Service + Ingress.”*

---

*Tài liệu tham khảo trong repo: `deploy/nginx/` (mẫu LB trong Docker), `deploy/nginx/host/` (Nginx OS HTTPS).*
