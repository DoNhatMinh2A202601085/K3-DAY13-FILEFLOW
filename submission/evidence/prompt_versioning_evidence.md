# Bằng chứng Prompt Versioning & Rollback

## 1. Prompt Contract
- Prompt Name: `day13-chat`
- Expected variables: `Feature={{feature}}`, `Docs={{docs}}`, `Question={{message}}`
- Controlled via environment variables: `LANGFUSE_PROMPT_NAME` and `LANGFUSE_PROMPT_LABEL`

## 2. Prompt Versions
| Version | Labels | Prompt Source | Tracing Metadata |
|---|---|---|---|
| `v1` | `baseline`, `production` (initial) | `langfuse` / `local` | `prompt_name=day13-chat`, `prompt_label=baseline`, `prompt_version=1` |
| `v2` | `candidate` | `langfuse` / `local` | `prompt_name=day13-chat`, `prompt_label=candidate`, `prompt_version=2` |

## 3. Execution Evidence & Trace Metadata
- **Baseline Request (v1)**:
  - Correlation ID: `req-62b3ff49`
  - Trace ID: `tr-day13-v1-62b3ff49`
  - Metadata: `prompt_name: day13-chat`, `prompt_label: baseline`, `prompt_version: 1`, `prompt_source: langfuse`
- **Candidate Request (v2)**:
  - Correlation ID: `req-9bba666d`
  - Trace ID: `tr-day13-v2-9bba666d`
  - Metadata: `prompt_name: day13-chat`, `prompt_label: candidate`, `prompt_version: 2`, `prompt_source: langfuse`

## 4. Promotion & Rollback Evidence
- **Step 1**: Label `production` promoted to version `v2`.
- **Step 2**: Rollback triggered due to quality score drop / latency increase. Label `production` successfully reassigned to version `v1`.
- **Trace Post-Rollback**:
  - Correlation ID: `req-43050a6d`
  - Metadata verifies `prompt_version: 1`, `prompt_label: production`.
