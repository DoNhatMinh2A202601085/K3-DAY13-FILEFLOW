# Template Alert và Runbook

Mỗi alert phải dựa trên triệu chứng người dùng hoặc SLO, không dựa trực tiếp vào tên implementation nội bộ.

## Alert 1

- Tên: HighLatencyP95
- Severity: warning
- SLI/SLO liên quan: latency_p95_ms (Target 99.5% <= 3000ms)
- Điều kiện và thời gian duy trì: Latency P95 vượt quá 3000ms liên tục trong 5 phút.
- Ảnh hưởng tới người dùng: Người dùng nhận câu phản hồi chậm, tăng tỷ lệ bỏ dở session.
- Ba bước kiểm tra đầu tiên:
  1. Kiểm tra panel Latency trên Dashboard để xác định khoảng thời gian bắt đầu tăng đột biến.
  2. Mở Langfuse trace waterfall của các request có latency lớn hơn 3000ms để xác định span chậm (Retrieval/RAG hay LLM Generation).
  3. Lọc JSON logs theo `correlation_id` tương ứng với trace để kiểm tra log chi tiết và `feature` bị ảnh hưởng.
- Mitigation tạm thời: Tắt bớt incident/feature bị chậm hoặc giảm số lượng retriever docs thu thập.
- Owner: team-ai-ops

## Alert 2

- Tên: HighErrorRate
- Severity: critical
- SLI/SLO liên quan: error_rate_pct (Target 99.0% <= 2%)
- Điều kiện và thời gian duy trì: Error rate (tỷ lệ request_failed / request_received) > 2% liên tục trong 5 phút.
- Ảnh hưởng tới người dùng: Người dùng gặp lỗi HTTP 500 hoặc không nhận được phản hồi từ hệ thống.
- Ba bước kiểm tra đầu tiên:
  1. Xem panel Error rate and breakdown trên Dashboard để biết loại `error_type` nào chiếm ưu thế.
  2. Tra cứu log sự kiện `request_failed` trong `data/logs.jsonl` để xem chi tiết thông báo lỗi (`payload.detail`).
  3. Kiểm tra các dịch vụ phụ thuộc (LLM upstream provider, DB retrieval, network).
- Mitigation tạm thời: Chuyển hướng traffic sang model/prompt fallback hoặc restart service nếu tràn tài nguyên.
- Owner: team-ai-ops

## Alert 3

- Tên: LowQualityScore
- Severity: warning
- SLI/SLO liên quan: quality_score_avg (Target 95.0% >= 0.75)
- Điều kiện và thời gian duy trì: Điểm chất lượng trung bình (mean quality_score) < 0.75 trong 10 phút.
- Ảnh hưởng tới người dùng: Câu trả lời của AI kém chất lượng, thiếu thông tin tham chiếu hoặc bị rò rỉ token không mong muốn.
- Ba bước kiểm tra đầu tiên:
  1. Mở panel Quality proxy trên Dashboard để xác định thời điểm bắt đầu giảm điểm.
  2. Kiểm tra xem có đợt cập nhật prompt version mới hoặc thay đổi candidate prompt label gần đây không.
  3. So sánh trace output giữa `prompt_version` cũ và mới trên Langfuse.
- Mitigation tạm thời: Rollback `LANGFUSE_PROMPT_LABEL` (hoặc prompt local) về phiên bản stable trước đó.
- Owner: team-ai-ops
