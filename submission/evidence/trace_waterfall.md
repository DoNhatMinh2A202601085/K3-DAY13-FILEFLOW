# Trace Waterfall Breakdown

## Normal Request Trace (Correlation ID: `req-62b3ff49`)
- Total Latency: `184ms`
- Span 1: `FastAPI /chat` (184ms)
  - Span 1.1: `retrieve` / RAG Document Retrieval (4ms)
  - Span 1.2: `resolve_prompt` (2ms)
  - Span 1.3: `FakeLLM.generate` (160ms)
  - Span 1.4: `_heuristic_quality` (2ms)

## Incident Request Trace (Correlation ID: `req-c1a35cf1` | Feature: `refund`)
- Total Latency: `2693.8ms` (Threshold: 2000ms - SLI Breached)
- Span 1: `FastAPI /chat` (2693.8ms)
  - **Span 1.1: `retrieve` / RAG Document Retrieval (2504.2ms) -> [BOTTLENECK IDENTIFIED]**
  - Span 1.2: `resolve_prompt` (1.8ms)
  - Span 1.3: `FakeLLM.generate` (172.5ms)
  - Span 1.4: `_heuristic_quality` (2.1ms)

### Conclusion from Trace Waterfall:
The latency spike is isolated to **Span 1.1 (`retrieve`)**, where RAG vector store document lookup took >2.5s due to the `rag_slow` incident condition. LLM Generation span remained normal (~172ms).
