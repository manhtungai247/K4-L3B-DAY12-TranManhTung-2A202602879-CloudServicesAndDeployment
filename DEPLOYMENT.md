# Thông Tin Deploy — Checkpoint 5

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Trần Mạnh Tùng |
| Mã học viên | 2A202602879 |
| Repo | https://github.com/manhtungai247/K4-L3B-DAY12-TranManhTung-2A202602879-CloudServicesAndDeployment |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://day12-agent-c12l.onrender.com |
| Platform | Render Blueprint |
| Ngày deploy | 2026-09-29 |
| Blueprint | `day12-agent` |
| Deploy commit | `fbc1bbb` |

## Biến Môi Trường Đã Set Trên Cloud

Chỉ ghi tên biến và nguồn cấp; không ghi giá trị secret.

| Biến | Đã set | Nguồn/Ghi chú |
|------|--------|---------------|
| `PORT` | ✅ | Render cấp tự động; log cho thấy Uvicorn bind `0.0.0.0:10000` |
| `AGENT_API_KEY` | ✅ | Nhập trong biểu mẫu Render Blueprint; giá trị không được lưu trong repo |
| `REDIS_URL` | ✅ | Nội suy từ Render Key Value `day12-redis` qua `connectionString` |
| `RATE_LIMIT_PER_MINUTE` | ✅ | Giá trị `10` đặt trong Blueprint |
| `MONTHLY_BUDGET_USD` | ✅ | Giá trị `10.0` đặt trong Blueprint |
| `LOG_LEVEL` | ✅ | Giá trị `INFO` đặt trong Blueprint |

## Lệnh Kiểm Tra

```bash
URL=https://day12-agent-c12l.onrender.com

curl -i "$URL/health"
curl -i "$URL/ready"
curl -i -X POST "$URL/ask" \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'
```

Để kiểm tra `/ask` có xác thực, đặt `DEPLOY_API_KEY` cục bộ rồi chạy request có header `X-API-Key`. Không ghi giá trị khóa vào file này.

## Kết Quả Deploy Và Kiểm Tra

- Render Blueprint đã đồng bộ từ nhánh `main`, commit `fbc1bbb`.
- Key Value `day12-redis` ở trạng thái **Available**; chế độ persistence là **Off** trên gói Free.
- Web service `day12-agent` ở trạng thái **Live**; deploy thành công trong `34.9s`.
- Log Render xác nhận ứng dụng khởi động hoàn tất trên `0.0.0.0:10000`; các health probe nội bộ gọi `/health` nhận `200 OK`.
- `pytest tests/test_cp5.py::TestDeploymentDoc -v`: `4 passed`.
- `pytest tests/test_cp5.py -v`: `5 passed, 3 failed, 5 skipped`; ba lỗi endpoint không tới ứng dụng vì môi trường này không phân giải được DNS (`getaddrinfo failed`). `curl.exe` cũng bị Windows chặn ở bước kiểm tra thu hồi chứng chỉ (`CRYPT_E_REVOCATION_OFFLINE`), và trình duyệt báo timeout. Các lỗi này chưa xác định được HTTP status của `/health`, `/ready` hay `/ask` từ client bên ngoài; log Render chỉ xác nhận health probe nội bộ `/health` trả `200`.
- Chạy lại ba lệnh HTTP trong PowerShell trên mạng của bạn để xác minh `/health` `200`, `/ready` `200` và `/ask` không key `401`. Không tắt kiểm tra TLS để vượt qua lỗi mạng.

## Ảnh Chụp Màn Hình

Lưu ảnh vào thư mục `screenshots/`:

- `screenshots/dashboard.png` — trang Render thể hiện service/deploy ở trạng thái Live.
- `screenshots/health.png` — kết quả gọi `https://day12-agent-c12l.onrender.com/health`.
