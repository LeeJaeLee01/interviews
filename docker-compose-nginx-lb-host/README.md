# Cách 1 — Docker Compose + Nginx (LB trong Docker) + Nginx trên host

Mẫu mang đi **phỏng vấn**: backend nhiều replica, **LB bằng Nginx trong Compose**, **HTTPS / domain** do **Nginx trên host** (hoặc máy chủ VPS) xử lý, proxy xuống `127.0.0.1:3030` (cổng do container **nginx-lb** publish).

## Kiến trúc (nói trong interview)

1. **Host Nginx**: TLS (Let’s Encrypt), `server_name`, `location /api/` → `proxy_pass http://127.0.0.1:3030`.
2. **Container `nginx-lb`**: publish `3030:80`, **upstream** tới service `app` (nhiều replica khi scale).
3. **Container `app`**: backend (demo dùng `http-echo` listen `:3030`); **không** `container_name`, **không** map trùng port ra host.

## Chạy nhanh

```bash
cd docker-compose-nginx-lb-host

# 1 replica (mặc định)
docker compose up -d

# 3 replica + LB
docker compose up -d --scale app=3
```

Kiểm tra qua LB (từ host):

```bash
curl -s http://127.0.0.1:3030/
```

## Gắn Nginx host

- Copy nội dung `host-nginx.example.conf` vào site thật (vd. `/etc/nginx/sites-available/...`), chỉnh `ssl_certificate`, `server_name`, đường dẫn `location`.
- `proxy_pass http://127.0.0.1:3030` trùng với `ports` của service `nginx-lb` trong `docker-compose.yml`.

## Điểm cần nói khi phỏng vấn

| Câu hỏi | Gợi ý trả lời |
|--------|----------------|
| Vì sao không publish `3030` trực tiếp cho 3 container `app`? | Trên một host chỉ một process listen một port; nhiều replica cùng port → xung đột. |
| Vai trò `nginx-lb`? | Một điểm vào duy nhất, `upstream` tới `app:3030`, scale `app` không đổi port host. |
| Nginx OSS có luôn round-robin đúng 3 IP của `app`? | DNS Docker có thể trả nhiều A record; Nginx mặc định resolve `app` lúc start — có thể chỉ thấy một IP. Production thường dùng **HAProxy** `server-template`, **Traefik**, hoặc **K8s Service**. |
| HTTPS ở đâu? | Trên **host** (certbot/Let’s Encrypt); trong Docker chỉ HTTP nội bộ tới LB (có thể bật TLS nội bộ nếu cần). |

## File trong thư mục

- `docker-compose.yml` — `app`, `nginx-lb`, network.
- `nginx-lb/default.conf` — upstream + proxy tới backend.
- `host-nginx.example.conf` — ví dụ HTTPS host → `127.0.0.1:3030`.
