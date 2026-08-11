# Báo cáo Day 13 Observability

## 1. Thông tin nhóm

- Tên nhóm: Team Observability K3
- Repository URL: https://github.com/K3-DAY13-FILEFLOW
- Commit SHA cuối: HEAD
- Thành viên và vai trò:
  - Trương Minh Hoàng (Người 1 - Logging & PII)
  - Bạn / User (Người 2 - Tracing & Prompt Version)
  - Đỗ Nhật Minh (Người 3 - Dashboard, SLO & Alert / Incident, Report & Demo)

## 2. Kết quả kỹ thuật

- Điểm `validate_logs.py`: 100/100 (Hợp lệ hoàn toàn)
- Tổng số traces: 34 log records / 15+ traces
- Số PII leak còn lại: 0
- Link/đường dẫn dashboard: `submission/evidence/validate_dashboard_result.txt` (6/6 panels valid)

## 3. Logging và tracing

- Evidence correlation ID: [log_correlation_id_sample.jsonl](file:///c:/Users/Admin/Documents/AIVin/K3-DAY13-FILEFLOW/submission/evidence/log_correlation_id_sample.jsonl)
- Evidence PII redaction: [pii_redaction_sample.jsonl](file:///c:/Users/Admin/Documents/AIVin/K3-DAY13-FILEFLOW/submission/evidence/pii_redaction_sample.jsonl)
- Evidence trace waterfall: [trace_waterfall.md](file:///c:/Users/Admin/Documents/AIVin/K3-DAY13-FILEFLOW/submission/evidence/trace_waterfall.md)
- Giải thích một span đáng chú ý: Span `retrieve` trong RAG document lookup bị nghẽn (duration 2504ms trên tổng 2693ms request) do sự cố `rag_slow`.

## 4. Prompt versioning

- Prompt name: `day13-chat`
- Version/label baseline: `v1` (`baseline`, `production`)
- Version/label candidate: `v2` (`candidate`)
- Trace ID của mỗi version:
  - Baseline (v1): `tr-day13-v1-62b3ff49` (Correlation ID: `req-62b3ff49`)
  - Candidate (v2): `tr-day13-v2-9bba666d` (Correlation ID: `req-9bba666d`)
- Bằng chứng đổi label hoặc rollback: [prompt_versioning_evidence.md](file:///c:/Users/Admin/Documents/AIVin/K3-DAY13-FILEFLOW/submission/evidence/prompt_versioning_evidence.md)

## 5. Dashboard, SLO và alerts

- Kết quả `validate_dashboard.py`: HỢP LỆ 6/6 panel (`submission/evidence/validate_dashboard_result.txt`)
- Evidence dashboard: 6 panels (Latency P50/P95/P99, Traffic, Error rate, Cost, Tokens in/out, Quality mean)
- SLO đã chọn và lý do:
  - Latency P95 <= 3000ms (tỉ lệ 99.5%): Bảo đảm trải nghiệm phản hồi mượt mà cho người dùng.
  - Error rate <= 2% (tỉ lệ 99.0%): Hạn chế gián đoạn dịch vụ AI API.
  - Quality score mean >= 0.75 (tỉ lệ 95.0%): Duy trì độ chuẩn xác của câu trả lời AI.
- Alert rules và runbook: [config/alert_rules.yaml](file:///c:/Users/Admin/Documents/AIVin/K3-DAY13-FILEFLOW/config/alert_rules.yaml) và [docs/alerts.md](file:///c:/Users/Admin/Documents/AIVin/K3-DAY13-FILEFLOW/docs/alerts.md)

## 6. Điều tra challenge

- Challenge ID: `day13-k3-observability-v1`
- Triệu chứng từ metrics: Latency P95 tăng vọt lên ~2660ms, vượt quá ngưỡng `latency_threshold_ms: 2000` của challenge ở feature `refund`.
- Trace ID liên quan: `req-c1a35cf1`
- Log line/correlation ID liên quan: `req-c1a35cf1` (`feature: refund`, `latency_ms: 2693`)
- Root cause: Incident `rag_slow` gây trễ 2.5 giây ở bước RAG Document Retrieval trong `mock_rag.py`.
- Fix action: Chạy `python scripts/inject_incident.py --disable` để tắt giả nhập sự cố `rag_slow`.
- Preventive measure: Đặt timeout 1000ms cho bước RAG retriever và tự động chuyển sang fallback document cache khi quá hạn.

## 7. Đóng góp cá nhân

Với mỗi thành viên, ghi rõ nhiệm vụ và link commit/PR tương ứng.

| Thành viên | Phần việc | Commit/PR | Điều đã học |
|---|---|---|---|
| Trương Minh Hoàng | Core Logging, Middleware Correlation ID, Metadata Enrichment & PII Redaction | HEAD | Hiểu sâu việc quản lý Correlation ID xuyên suốt middleware và quy trình khử PII trong JSON logging. |
| Bạn (Người 2) | Tracing Metadata, Prompt Versioning (v1/v2), Promotion & Rollback Evidence | HEAD | Nắm vững cách quản lý lifecycle của Prompt, theo dõi Span trên Langfuse và quy trình Rollback. |
| Đỗ Nhật Minh | Dashboard Setup (6 panels), SLO Definition, Alert Rules, Runbook & Challenge Report | HEAD | Thành thạo cách đọc metrics trên Dashboard để nối với Traces và chứng minh Root cause từ Logs. |


