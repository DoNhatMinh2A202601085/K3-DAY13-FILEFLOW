# PHÂN CÔNG CÔNG VIỆC — DAY 13 OBSERVABILITY

> Mỗi checkpoint đánh dấu thứ tự bắt buộc. Không sang checkpoint tiếp theo khi chưa hoàn thành checkpoint hiện tại. Git push theo đúng thứ tự trong bảng để tránh conflict.

---

## TỔNG QUAN VAI TRÒ

| Người | Vai chính | Vai phụ (hỗ trợ khi cần) |
|---|---|---|
| **Người 1** | Logging & PII | Hỗ trợ Trace/Prompt Version |
| **Người 2** | Tracing & Prompt Version | Hỗ trợ Logging & PII |
| **Người 3** | Dashboard, SLO & Alert | Incident, Report & Demo |

**Lưu ý:** Người 3 giữ vai Incident/Report/Demo vì đây là vai tiêu thụ output của cả 2 người còn lại — cần hiểu toàn cảnh nhất. Người 1 và Người 2 hỗ trợ chéo nhau trong CP2.

---

## CHECKPOINT 0 — 0:00–0:30: Setup và baseline

**Mục tiêu:** Môi trường chạy, mọi người hiểu repo.

| Thứ tự | Ai làm | Làm gì | Git |
|---|---|---|---|
| 1 | **Người 2** | Clone repo, làm theo SETUP.md, cấu hình Langfuse chung/cloud, chạy API `uvicorn app.main:app --reload --env-file .env` | Không commit gì giai đoạn này |
| 2 | **Người 1** | Chạy `python scripts/load_test.py` (terminal riêng), kiểm tra `data/logs.jsonl` có dữ liệu | — |
| 3 | **Người 3** | Đọc `config/dashboard.yaml`, `config/log_schema.yaml`, ghi chú các TODO cần làm trong dashboard và alert | — |
| 4 | **Tất cả** | Chạy baseline: `python scripts/validate_logs.py` và `python scripts/validate_dashboard.py` | — |

**Người 3 ghi baseline** `validate_logs.py` vào báo cáo (phần checkpoint 0). **Không push gì lên git giai đoạn setup** — tránh conflict sau này.

---

## CHECKPOINT 1 — 0:30–1:30: Logging và PII

**Mục tiêu:** Correlation ID, metadata đầy đủ, PII đã che, `validate_logs.py` ≥ 80/100.

**File chính cần sửa:** `app/` và `config/log_schema.yaml`

| Thứ tự | Ai làm | Làm gì | Git |
|---|---|---|---|
| 1 | **Người 1** | Hoàn thiện correlation ID, thêm metadata (`user_id_hash`, `session_id`, `feature`, `model`, `env`) vào log | — |
| 2 | **Người 1** | Triển khai PII redaction (email, phone, card) trong `app/logging/` hoặc `config/` | — |
| 3 | **Người 2** | Hỗ trợ kiểm tra log có đúng schema, chạy `python scripts/validate_logs.py` sau mỗi thay đổi lớn | — |
| 4 | **Người 1** | Đạt ≥ 80/100 → chụp ảnh log với correlation ID và bằng chứng PII đã che → lưu vào `submission/evidence/` | — |
| 5 | **Người 1** | Commit: `app/`, `config/log_schema.yaml`, evidence → push lên branch cá nhân | **Push sau khi Người 2 hoàn tất CP1 phần mình (bước 2 bên dưới)** |

**Quy tắc git CP1:**
- Người 1 commit xong **chờ Người 2 push trước**, rồi mới push. Lý do: Người 2 cần sửa `app/tracing/` — tránh conflict nếu cùng chạm `app/main.py`.
- Thứ tự push: **Người 2 → Người 1**. Nếu Người 1 push trước mà có conflict, ưu tiên giữ code của Người 1 (vì Người 2 chỉ hỗ trợ, không sửa logging).

---

## CHECKPOINT 2 — 1:30–2:30: Metrics, traces, prompt version và dashboard

**Mục tiêu:** 10 traces, prompt v1/v2, rollback, validator 6/6 panel, SLO, alert.

**File chính cần sửa:** `app/tracing/`, `docs/`, `config/`, `app/` (prompt), `submission/evidence/`

| Thứ tự | Ai làm | Làm gì | Git |
|---|---|---|---|
| 1 | **Người 2** | Triển khai tracing: thêm metadata vào traces, chạy `load_test.py` để tạo ≥ 10 traces trên Langfuse | — |
| 2 | **Người 2** | Làm prompt versioning theo `docs/PROMPT_VERSIONING.md`: tạo prompt v1 và v2, gắn label + version metadata | — |
| 3 | **Người 2** | Thực hiện 1 lần đổi label hoặc rollback → chụp ảnh Langfuse (trace trước/sau, thao tác rollback) → lưu vào `submission/evidence/` | Commit: `app/tracing/`, prompt config → push (sau Người 1 đã push CP1) |
| 4 | **Người 1** | (Song song với bước 1-3 của Người 2) Tiếp tục fix `validate_logs.py` nếu chưa đạt 80 | — |
| 5 | **Người 3** | Triển khai dashboard 6 panel theo `docs/DASHBOARD_SETUP.md` và `config/dashboard.yaml` từ `data/logs.jsonl` | — |
| 6 | **Người 3** | Thêm SLO line/threshold, viết alert rule và runbook đơn giản | — |
| 7 | **Người 3** | Chạy `python scripts/validate_dashboard.py` → phải ra `6/6 panel` → chụp ảnh dashboard + kết quả validator → lưu vào `submission/evidence/` | Commit: `config/dashboard.yaml`, alert config, evidence → push (sau khi Người 2 push) |
| 8 | **Người 2** | Sau khi dashboard xong, kiểm tra trace hiển thị đúng `prompt_name`, `prompt_label`, `prompt_version` | — |
| 9 | **Người 1** | Merge code từ Người 2 và Người 3, chạy lại `validate_logs.py` để xác nhận không bị regress | — |

**Quy tắc git CP2:**
- **Thứ tự push: Người 2 → Người 3 → Người 1 merge và push cuối.**
- Người 2 push trước vì code tracing/prompt ảnh hưởng đến dashboard (metric source). Người 3 cần push sau để dashboard config không bị ghi đè. Người 1 push cuối để merge và kiểm tra tổng hợp.
- Nếu có conflict trong `app/main.py`: ưu tiên giữ logic logging của Người 1, thêm logic tracing của Người 2 (hai người tự trao đổi để merge đúng).

---

## CHECKPOINT 3 — 2:30–3:30: Challenge chính thức

**Mục tiêu:** Xác định root cause, nối Metrics → Traces → Logs, đề xuất fix.

> **Chỉ chạy sau khi Lab Coach release `config/challenge.json`.**

| Thứ tự | Ai làm | Làm gì | Git |
|---|---|---|---|
| 1 | **Người 3** | Chạy challenge: `python scripts/inject_incident.py` và `python scripts/load_test.py --challenge --concurrency 5` | — |
| 2 | **Người 3** | Đọc metrics → xác định triệu chứng → tra dashboard để xem panel nào bất thường | — |
| 3 | **Người 2** | Dùng trace ID từ Người 3 để khoanh vùng span bất thường trên Langfuse | — |
| 4 | **Người 1** | Dùng correlation ID / trace ID để đọc log → chứng minh root cause | — |
| 5 | **Người 3** | Tổng hợp: ghi root cause, fix action và preventive measure vào file tạm | — |
| 6 | **Người 3** | Commit evidence challenge (log đoạn root cause, ảnh trace, ảnh metrics) → push | Commit sau cùng, sau khi Người 1 và 2 đã push evidence |
| 7 | **Người 1 + Người 2** | Gửi evidence cho Người 3 tổng hợp vào report | — |

**Quy tắc git CP3:**
- **Người 1 và Người 2 commit và push evidence của mình trước.**
- Người 3 commit tổng hợp và push cuối cùng. Không ai được push sau Người 3 trong checkpoint này.
- Nếu `config/challenge.json` chưa release: **không làm gì thêm, chờ.** Không tự tạo hoặc sửa file này.

---

## CHECKPOINT HOÀN TẤT — 3:30–4:00: Báo cáo, demo và nộp bài

**Mục tiêu:** Report hoàn chỉnh, không có secret/PII trong git, commit cuối cùng.

| Thứ tự | Ai làm | Làm gì | Git |
|---|---|---|---|
| 1 | **Người 1** | Viết phần Logging & PII trong `submission/REPORT.md` | — |
| 2 | **Người 2** | Viết phần Tracing & Prompt Version trong `submission/REPORT.md` | — |
| 3 | **Người 3** | Viết phần Dashboard, SLO, Alert và Incident trong `submission/REPORT.md` | — |
| 4 | **Người 3** | Chạy `python -m pytest -q` → xác nhận pass | — |
| 5 | **Người 1** | Kiểm tra `.env`, API key, PII không nằm trong git: `git log`, `git diff HEAD~1` | — |
| 6 | **Người 2** | Kiểm tra `submission/` đầy đủ: report + evidence có đủ ảnh, trace ID, log đoạn root cause | — |
| 7 | **Người 1** | Commit cuối cùng: `submission/`, report → push → dán commit SHA vào report | **Commit và push cuối cùng của cả lab** |
| 8 | **Người 3** | Chuẩn bị demo: luồng Metrics → Traces → Logs → Root cause (tối đa 3 phút) | — |

**Quy tắc git hoàn tất:**
- **Người 1 push commit cuối cùng.** Lý do: Người 1 làm phần đầu tiên và đã merge tổng hợp ở CP2, phù hợp làm commit cuối để đảm bảo không ai ghi đè.
- Sau commit cuối, **không ai push thêm** cho đến khi demo xong.
- Gửi repo URL + commit SHA cho Lab Coach.

---

## TỔNG HỢP THỨ TỰ PUSH GIT

```
CP1: Người 2 push → Người 1 push
CP2: Người 2 push → Người 3 push → Người 1 merge & push
CP3: Người 1 push evidence → Người 2 push evidence → Người 3 push tổng hợp
CP4: Người 1 push commit cuối cùng
```

**Nguyên tắc chung:**
- Mỗi người làm việc trên branch riêng hoặc thư mục riêng nếu cùng sửa một file.
- Nếu cùng sửa `app/main.py` hoặc `app/logging/` và `app/tracing/`: trao đổi trước khi code, code xong merge tay rồi mới push.
- Không push khi người phía sau trong thứ tự chưa hoàn tất việc.

---

## BẢNG EVIDENCE PHẢI CÓ

| Checkpoint | Người chịu trách nhiệm | File evidence |
|---|---|---|
| CP0 | Người 3 | Ảnh health endpoint + baseline validator |
| CP1 | Người 1 | Log có correlation ID + ảnh PII đã che |
| CP2 | Người 2 | 2 trace ID + ảnh prompt label/rollback + ảnh validator 6/6 |
| CP2 | Người 3 | Ảnh dashboard + SLO + alert + validator |
| CP3 | Người 3 | Log đoạn root cause + ảnh trace + ảnh metrics |
| CP4 | Người 3 | Demo chạy thành công (trình chiếu trong buổi) |

---

## LIÊN HỆ KHI CẦN HỖ TRỢ

| Tình huống | Ai hỗ trợ |
|---|---|
| Log không ra đúng schema | Người 1 hỗ trợ Người 2 |
| Trace không hiển thị metadata | Người 2 hỗ trợ Người 1 |
| Dashboard panel không khớp logs | Người 1 + Người 2 hỗ trợ Người 3 |
| Challenge không chạy được | Cả nhóm debug cùng lúc |
| Git conflict nghiêm trọng | Cả nhóm dừng 5 phút giải quyết |
