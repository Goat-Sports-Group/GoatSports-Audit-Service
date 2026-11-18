# GoatSports Audit Service

Nhật ký hoạt động toàn hệ thống và giám sát vận hành cho quản trị viên.

## Chức năng chính

- **Thu nhật ký**: mọi service gắn `@AuditLog` (thư viện shared-kernel) phát sự kiện lên Kafka `user-log`;
  `AuditLogConsumer` lưu vào MongoDB kèm người thực hiện, correlation ID, thay đổi trước / sau. Khử trùng theo `eventId`.
- **Nhật ký thanh toán**: `PaymentAuditConsumer` ghi lại sự kiện từ `payment.events`.
- **Chống sửa nhật ký**: các bản ghi được nối **chuỗi băm SHA-256** (mỗi bản ghi chứa mã băm của bản ghi trước);
  `/api/v1/logs/integrity` kiểm tra chuỗi còn nguyên vẹn hay không.
- **Giám sát hệ thống** (`/api/v1/admin/system`): đọc chỉ số từ Prometheus, tình trạng các service;
  `MonitoringScheduler` phát hiện sự cố và báo cho admin qua notification-service.

## API chính

| Đường dẫn | Mô tả |
|---|---|
| `/api/v1/logs` | Tra cứu, lọc nhật ký; kiểm tra toàn vẹn |
| `/api/v1/admin/system` | Giám sát, sự cố |

## Cấu trúc

```
src/main/java/iuh/fit/auditservice/
├── domain/          # Log, AuditChange, AuditContext, Incident
├── application/     # service, port, DTO, mapper
├── infrastructure/  # MongoDB, Kafka consumer, client Prometheus / Grafana
└── presentation/    # REST controller
```

## Tích hợp

- **MongoDB** (nhật ký), **Redis**.
- **Kafka nhận**: `user-log`, `payment.events`.
- Gọi Prometheus / Grafana (`PROMETHEUS_URL`, `GRAFANA_URL`) và notification-service.

## Chạy và kiểm thử

```bash
mvn spring-boot:run -Dspring-boot.run.profiles=dev     # cần application-dev.properties (bị .gitignore)
mvn test
```

Container cần `TZ=Asia/Ho_Chi_Minh` vì thời gian nhật ký lưu theo giờ địa phương.

## CI/CD

`.gitlab-ci.yml`: test → `jib:build` lên Docker Hub → gọi Jenkins job `GoatSports-CD`.
