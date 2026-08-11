# Báo cáo Điều tra Challenge (`day13-k3-observability-v1`)

## 1. Challenge Overview
- **Cohort**: K3
- **Challenge ID**: `day13-k3-observability-v1`
- **Affected Feature**: `refund`
- **Latency Threshold**: `2000ms`

## 2. Luồng Điều tra Observability (Metrics -> Traces -> Logs)

### Bước 1: Phát hiện từ Metrics
- Panel `Latency percentiles` báo latency P95 tăng vọt từ ~200ms lên **2660ms**, vượt xa ngưỡng threshold 2000ms / 3000ms SLO.
- Mức độ ảnh hưởng: Chỉ diễn ra ở feature `refund`.

### Bước 2: Khoanh vùng bằng Traces
- Mở trace của request thuộc feature `refund` (Trace ID / Correlation ID: `req-c1a35cf1`).
- Waterfall analysis cho thấy:
  - Total Duration: 2693.8ms
  - `retrieve` span duration: 2504.2ms (chiếm 93% tổng thời gian request).
  - LLM generation span duration: 172.5ms (bình thường).
- **Kết luận từ Trace**: Nút thắt cổ chai nằm ở công đoạn RAG Document Retrieval.

### Bước 3: Chứng minh Root Cause từ Logs
- Tra cứu JSON logs trong `data/logs.jsonl` với `correlation_id: req-c1a35cf1`:
  ```json
  {"service": "api", "payload": {"message_preview": "What is your refund policy?"}, "event": "request_received", "env": "dev", "feature": "refund", "session_id": "k3-challenge-s01", "correlation_id": "req-c1a35cf1", "model": "claude-sonnet-4-5", "user_id_hash": "1d8cfbb182f7", "level": "info", "ts": "2026-08-11T03:41:14.283115Z"}
  {"service": "api", "latency_ms": 2693, "tokens_in": 37, "tokens_out": 154, "cost_usd": 0.002421, "quality_score": 0.9, "payload": {"answer_preview": "Starter answer. Teams should improve this output logic and add better quality ch..."}, "event": "response_sent", "env": "dev", "feature": "refund", "session_id": "k3-challenge-s01", "correlation_id": "req-c1a35cf1", "model": "claude-sonnet-4-5", "user_id_hash": "1d8cfbb182f7", "level": "info", "ts": "2026-08-11T03:41:16.977412Z"}
  ```
- Kết hợp thông tin trạng thái control route: `STATE["rag_slow"] = True` gây trễ 2.5s trong `mock_rag.retrieve()`.

## 3. Action Plan & Preventive Measures
- **Root Cause**: Cấu hình truy vấn RAG retriever bị hoãn/chậm (latency injection / database connection pool latency ở dịch vụ vector db đối với feature refund).
- **Fix Action**: Tắt incident toggle `rag_slow` (`python scripts/inject_incident.py --disable`) và khôi phục connection pool.
- **Preventive Measure**:
  - Thêm timeout ngắn (ví dụ: 1000ms) cho bước RAG retrieval đi kèm fallback strategy (chạy với local cache document hoặc general prompt) khi vector db bị trễ.
  - Thiết lập Alert `HighLatencyP95` cảnh báo ngay khi P95 > 3000ms duy trì trong 5 phút.
