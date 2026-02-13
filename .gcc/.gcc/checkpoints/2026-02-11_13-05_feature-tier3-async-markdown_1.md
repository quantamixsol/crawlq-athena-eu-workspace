# Checkpoint: feature-tier3-async-markdown — COMMIT 1

**Date:** 2026-02-11T13:05:52Z
**Branch:** feature-tier3-async-markdown
**Commit:** 1
**State:** WORKING

---

## Milestone

Comprehensive Tier 3 architecture designed — ADR-012, infrastructure script, implementation guide

---

## Summary

Designed complete enterprise-grade async architecture to eliminate 503 timeout errors and provide Claude Code-level markdown rendering quality. Created comprehensive ADR-012 (23KB architecture document), infrastructure provisioning script (DynamoDB + S3 + SQS + IAM), and full implementation guide with deployment steps.

### Architecture Highlights

**Problem Solved:**
- API Gateway 30s timeout → 503 errors on complex queries
- Poor output formatting → tables/charts render badly
- No response persistence → lost on refresh
- No progress feedback → users wait blindly

**Solution:**
- Async job queue (SQS) with 15-minute Lambda workers
- Intelligent markdown processor (table/chart/diagram detection)
- S3 persistence (every response as markdown file)
- 5-stage progress indicators (RAG → graph → reasoning → formatting)
- Enterprise React renderer (GFM tables, mermaid diagrams, syntax highlighting)

**Components:**
1. **EUChatJobQueue** — Job submission endpoint
2. **EUChatJobWorker** — Background processor (SQS trigger)
3. **EUChatJobStatus** — Polling endpoint
4. **IntelligentMarkdownProcessor** — Python module for formatting
5. **EnterpriseMarkdownRenderer** — React component
6. **JobProgressIndicator** — Progress UI component
7. **useJobPolling** — React Query hook

---

## Files Changed

### Created

**`.gsm/decisions/ADR-012-tier3-async-markdown-architecture.md` (23KB)**
- Complete architecture decision record
- Detailed diagrams of async flow
- Lambda function specifications
- Database schemas (DynamoDB, S3)
- Intelligent markdown processor design
- Frontend component specifications
- API Gateway route definitions
- Deployment phases (7 days)
- Cost estimation ($525-1040/month)
- Success criteria checklist

**`provision_tier3_infrastructure.py`**
- DynamoDB table creation (`eu_chat_jobs` with GSIs)
- S3 bucket creation (`crawlq-eu-chat-responses` with lifecycle)
- SQS queue creation (`eu-chat-jobs` with 15m visibility)
- IAM role policy updates (S3 + SQS + DynamoDB permissions)
- Automated provisioning with error handling

**`TIER3_IMPLEMENTATION_GUIDE.md` (12KB)**
- Executive summary with metrics table
- Architecture flow diagram
- Infrastructure component specifications
- Lambda function code examples
- Frontend component code examples
- Deployment steps (5 phases, 7 days)
- Monitoring and alerts setup
- Cost breakdown
- Security considerations
- Rollback plan
- Success criteria checklist

### Modified

**`.gcc/branches/feature-tier3-async-markdown/commit.md`**
- Added COMMIT 1 block

**`.gcc/branches/feature-tier3-async-markdown/metadata.yaml`**
- Updated file_tree with 3 new files

**`.gcc/registry.md`**
- Added feature-tier3-async-markdown branch entry

---

## Key Decisions

### 1. Async Architecture (SQS + Lambda Workers)
**Decision:** Use SQS queue with 15-minute Lambda workers instead of API Gateway direct invocation
**Rationale:**
- API Gateway has hard 30s timeout (cannot be changed)
- Lambda supports up to 15 minutes timeout
- SQS provides reliable async message delivery
- Allows horizontal scaling of workers
- Job status tracked in DynamoDB

### 2. Markdown Persistence (S3)
**Decision:** Save every response as markdown file in S3 with YAML frontmatter
**Rationale:**
- Audit trail for compliance (GDPR, EU AI Act)
- Users can download/share responses
- Presigned URLs for secure access
- Lifecycle policies (archive to Glacier, expire after 1 year)
- Metadata stored in object headers

### 3. Intelligent Markdown Processor
**Decision:** Custom Python module to detect and format tables, charts, diagrams
**Rationale:**
- Claude outputs vary in format (pipe tables, aligned columns, ASCII boxes)
- Need consistent GFM formatting for react-markdown
- Auto-generate mermaid diagrams from text descriptions
- Render charts as ASCII art or mermaid
- Add YAML frontmatter with metadata

### 4. Progress Tracking (5 Stages)
**Decision:** Track job status through 5 distinct stages with progress percentage
**Rationale:**
- Users need feedback during long processing (30s-5min)
- Each stage maps to architectural component (RAG, KG, Claude, formatting)
- Progress bar provides visual reassurance
- Estimated time remaining calculated from stage
- Frontend polls every 2s (balance between UX and API calls)

### 5. Enterprise Markdown Renderer
**Decision:** React component with remark-gfm, rehype-mermaid, syntax highlighting
**Rationale:**
- Match Claude Code rendering quality
- GFM tables with styling (borders, headers, alternating rows)
- Mermaid diagrams (flowcharts, sequence, Gantt)
- Prism.js syntax highlighting for code blocks
- Callout boxes (info, warning, success)
- LaTeX math support (KaTeX)

### 6. Database Schema
**Decision:** DynamoDB with composite key (job_id + timestamp) and 2 GSIs
**Rationale:**
- Fast job lookups by job_id
- User history via user-timestamp-index GSI
- Monitor stuck jobs via status-index GSI
- TTL auto-deletes after 24h (GDPR data minimization)
- Pay-per-request billing (no capacity planning)

### 7. S3 Lifecycle Policy
**Decision:** Intelligent-Tiering (30d) → Glacier (90d) → Expire (365d)
**Rationale:**
- Recent responses (30d) need fast access
- Old responses (90d+) rarely accessed → cheaper storage
- 1-year retention for audit/compliance
- Auto-delete after 1 year (GDPR right to erasure)

---

## Specifications

### Lambda Functions

**EUChatJobQueue:**
- Timeout: 30s
- Memory: 256 MB
- Trigger: API Gateway POST /chat-async
- Output: job_id, status_url, estimated_time

**EUChatJobWorker:**
- Timeout: 900s (15 minutes)
- Memory: 1024 MB
- Trigger: SQS eu-chat-jobs
- Batch size: 1 message
- Processing: RAG → KG → Claude → Markdown → S3

**EUChatJobStatus:**
- Timeout: 10s
- Memory: 256 MB
- Trigger: API Gateway GET /chat-status/:id
- Output: status, progress, s3_url, error, estimated_completion

### DynamoDB Schema

```yaml
Table: eu_chat_jobs
PK: job_id (String)
SK: timestamp (String)
Attributes:
  status: pending|rag_retrieval|graph_building|reasoning|formatting|completed|failed
  progress: Number (0-100)
  s3_url: String
  s3_key: String
  error: String
  user_id: String
  username: String
  workspace: String
  question: String
  answer: String (truncated 400 chars)
  model: String
  confidence_score: Number
  ttl: Number (Unix timestamp)
GSI 1: user-timestamp-index (PK: user_id, SK: timestamp)
GSI 2: status-index (PK: status, SK: timestamp)
```

### S3 Bucket

```yaml
Bucket: crawlq-eu-chat-responses
Region: eu-central-1
Encryption: AES256
Versioning: Enabled
Object Key: responses/{year}/{month}/{job_id}.md
Metadata:
  x-amz-meta-user-id
  x-amz-meta-username
  x-amz-meta-model
  x-amz-meta-confidence
  x-amz-meta-job-id
Lifecycle:
  - 30d: Intelligent-Tiering
  - 90d: Glacier
  - 365d: Expire
```

### Markdown Format

```markdown
---
title: "{query response title}"
query: "{user question}"
model: "eu.anthropic.claude-opus-4-6-v1"
timestamp: "2026-02-11T13:05:52Z"
job_id: "job_20260211_130552_abc123"
user_id: "user-sub-uuid"
username: "user@example.com"
workspace: "default"
confidence_score: 0.92
confidence_tier: "HIGH"
processing_time: 127.5
sections:
  - type: "executive_summary"
    lines: 1-25
  - type: "detailed_analysis"
    lines: 26-180
charts: 3
tables: 8
diagrams: 2
code_blocks: 0
---

# Response Title

## Section 1

| Column 1 | Column 2 | Column 3 |
|----------|----------|----------|
| Value 1  | Value 2  | Value 3  |

### Subsection

```
Chart ████████████████████ 80%
```

```mermaid
flowchart LR
    A[Start] --> B[Process]
    B --> C[End]
```
```

### Frontend Dependencies

```json
{
  "react-markdown": "^10.0.0",
  "remark-gfm": "^4.0.0",
  "remark-math": "^6.0.0",
  "rehype-katex": "^7.0.0",
  "rehype-mermaid": "^2.1.0",
  "rehype-raw": "^7.0.0",
  "react-syntax-highlighter": "^15.5.0",
  "katex": "^0.16.9",
  "mermaid": "^10.9.0"
}
```

### Cost Estimate

- DynamoDB: $1-5/month (1M requests)
- S3: $10-20/month (500 GB)
- SQS: $1-2/month
- Lambda: $10.70/month (10K queries)
- Bedrock: $500-1000/month (10K queries, Opus 4.6)
- **Total: $525-1040/month**

---

## Next Steps

- [ ] Run `python provision_tier3_infrastructure.py` to create AWS resources
- [ ] Build EUChatJobQueue Lambda function
- [ ] Build EUChatJobWorker Lambda function with IntelligentMarkdownProcessor
- [ ] Build EUChatJobStatus Lambda function
- [ ] Create EnterpriseMarkdownRenderer React component
- [ ] Create JobProgressIndicator React component
- [ ] Create useJobPolling React Query hook
- [ ] Update API Gateway with new routes
- [ ] Install frontend npm dependencies
- [ ] Update ChatContainer to use async flow
- [ ] Test with complex 5-minute query
- [ ] Verify markdown rendering matches Claude Code quality

---

## Blockers

None

---

## Session Log

### Actions
- [13:00] Branch created from feature-eu-chat-athena
- [13:05] Designed complete Tier 3 architecture (ADR-012)
- [13:15] Created infrastructure provisioning script
- [13:25] Created comprehensive implementation guide
- [13:35] Updated GCC branch files and registry

### Files Touched
- CREATED: `.gsm/decisions/ADR-012-tier3-async-markdown-architecture.md`
- CREATED: `provision_tier3_infrastructure.py`
- CREATED: `TIER3_IMPLEMENTATION_GUIDE.md`
- MODIFIED: `.gcc/branches/feature-tier3-async-markdown/commit.md`
- MODIFIED: `.gcc/branches/feature-tier3-async-markdown/metadata.yaml`
- MODIFIED: `.gcc/registry.md`

### Summary

Designed comprehensive Tier 3 enterprise-grade async architecture that eliminates all 503 timeout errors and provides Claude Code-level markdown rendering quality. Created ADR-012 (23KB architecture spec), infrastructure provisioning script (DynamoDB + S3 + SQS + IAM), and full implementation guide with deployment steps. Architecture uses SQS job queue with 15-minute Lambda workers, intelligent markdown processor, S3 persistence, 5-stage progress tracking, and enterprise React renderer with GFM tables, mermaid diagrams, and syntax highlighting. Ready for implementation.
