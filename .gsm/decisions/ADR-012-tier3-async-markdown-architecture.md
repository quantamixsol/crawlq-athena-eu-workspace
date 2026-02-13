# ADR-012: Tier 3 Enterprise-Grade Async Architecture with Intelligent Markdown Processing

**Date:** 2026-02-11
**Status:** ACCEPTED
**Context:** Eliminate 503 errors, provide properly formatted structured output like Claude Code

---

## Problem Statement

### Current Issues
1. **503 Timeout Errors:** API Gateway 30s hard timeout causes failures on complex queries
2. **Poor Output Formatting:** Tables, charts, diagrams render poorly in UI
3. **No Structured Output:** Response is plain text, lacks semantic structure
4. **No Response Persistence:** Responses not saved as markdown files for audit/history
5. **No Progress Indicators:** Users don't know what's happening during long queries
6. **Suboptimal UX:** Manual refresh required, no intermediate feedback

### User Experience Requirements
- No timeouts regardless of query complexity (up to 15 minutes)
- Rich formatted output: tables, charts, diagrams, code blocks
- Progress indicators showing RAG retrieval, graph building, reasoning
- Every response saved as markdown file with metadata
- Claude Code-level rendering quality

---

## Architecture Decision

### Tier 3: Enterprise-Grade Async Job Queue Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│                         CLIENT (Next.js)                              │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │  1. Submit Query → POST /chat-async                            │  │
│  │  2. Receive job_id + status_url                                │  │
│  │  3. Poll /chat-status/{job_id} every 2s                        │  │
│  │  4. Render progress indicators                                 │  │
│  │  5. Fetch markdown when status=completed                       │  │
│  │  6. Rich render with react-markdown + plugins                  │  │
│  └────────────────────────────────────────────────────────────────┘  │
└───────────────────────────┬──────────────────────────────────────────┘
                            │
                            ▼
┌──────────────────────────────────────────────────────────────────────┐
│                       API GATEWAY HTTP API                            │
│  Routes:                                                              │
│    POST   /chat-async      → EUChatJobQueue (submit job)             │
│    GET    /chat-status/:id → EUChatJobStatus (poll status)           │
│    GET    /chat-result/:id → EUChatJobResult (fetch markdown)        │
└───────────────────────────┬──────────────────────────────────────────┘
                            │
        ┌───────────────────┴────────────────────┐
        ▼                                        ▼
┌────────────────────┐                  ┌────────────────────┐
│  EUChatJobQueue    │                  │  EUChatJobStatus   │
│  Lambda            │                  │  Lambda            │
│                    │                  │                    │
│  - Validate input  │                  │  - Read job status │
│  - Generate job_id │                  │  - Return progress │
│  - Write to        │                  │  - Return S3 URL   │
│    DynamoDB        │                  │                    │
│  - Push to SQS     │                  └────────────────────┘
│  - Return job_id   │
└────────┬───────────┘
         │ Push message to SQS
         ▼
┌────────────────────┐
│   Amazon SQS       │
│   eu-chat-jobs     │
│   - Visibility: 15m│
│   - Retention: 1h  │
└────────┬───────────┘
         │ Pull message
         ▼
┌────────────────────────────────────────────────────────────────────┐
│                     EUChatJobWorker Lambda                          │
│  (Triggered by SQS, 15-minute timeout)                             │
│                                                                     │
│  Step 1: Update status = "rag_retrieval"                           │
│  Step 2: Fetch RAG chunks from vector DB                           │
│  Step 3: Update status = "graph_building"                          │
│  Step 4: Build knowledge graph entities                            │
│  Step 5: Update status = "reasoning"                               │
│  Step 6: Invoke Bedrock with structured prompt                     │
│  Step 7: Update status = "formatting"                              │
│  Step 8: Process response → Intelligent Markdown Processor         │
│  Step 9: Save markdown to S3 + DynamoDB                            │
│  Step 10: Update status = "completed"                              │
└─────────────────────────────┬──────────────────────────────────────┘
                              │
                              ▼
                    ┌──────────────────────────┐
                    │  EUMarkdownProcessor     │
                    │  (Module/Lambda)         │
                    │                          │
                    │  - Parse Claude response │
                    │  - Detect tables         │
                    │  - Format charts (ASCII) │
                    │  - Add mermaid diagrams  │
                    │  - Structure sections    │
                    │  - Add metadata header   │
                    │  - Generate YAML frontm. │
                    └──────────┬───────────────┘
                               │
        ┌──────────────────────┴─────────────────────────┐
        ▼                                                 ▼
┌────────────────────┐                          ┌────────────────────┐
│  DynamoDB          │                          │  S3 Bucket         │
│  eu_chat_jobs      │                          │  eu-chat-responses │
│                    │                          │                    │
│  PK: job_id        │                          │  Path:             │
│  SK: timestamp     │                          │  /responses/       │
│  status:           │                          │    {year}/{month}/ │
│    - pending       │                          │    {job_id}.md     │
│    - rag_retrieval │                          │                    │
│    - graph_building│                          │  Metadata:         │
│    - reasoning     │                          │  - Content-Type    │
│    - formatting    │                          │  - x-amz-meta-user │
│    - completed     │                          │  - x-amz-meta-model│
│    - failed        │                          │  - x-amz-meta-conf │
│  progress: 0-100   │                          │                    │
│  s3_url: string    │                          │  Lifecycle:        │
│  error: string     │                          │  - Archive 90 days │
│  ttl: 24h          │                          │  - Delete 1 year   │
└────────────────────┘                          └────────────────────┘
```

---

## Intelligent Markdown Processor

### Design Philosophy
**Goal:** Match Claude Code rendering quality — structured, semantic, visually rich

### Processing Pipeline

```python
class IntelligentMarkdownProcessor:
    """
    Enterprise-grade markdown processor for Claude responses.

    Capabilities:
    - Table detection and formatting (GFM tables)
    - ASCII chart rendering (bar charts, timelines, flowcharts)
    - Mermaid diagram generation from text descriptions
    - Section structuring (h1, h2, h3 hierarchy)
    - Code block language detection
    - Callout boxes (info, warning, success)
    - YAML frontmatter (metadata)
    - LaTeX math (inline and block)
    - Footnotes and citations
    - Interactive elements (checkboxes, collapsible sections)
    """

    def process(self, claude_response: str, metadata: dict) -> MarkdownDocument:
        """
        Main processing pipeline.

        Input: Raw Claude response
        Output: Structured markdown document with frontmatter
        """
        # Step 1: Parse response structure
        sections = self._detect_sections(claude_response)

        # Step 2: Detect and format tables
        sections = self._format_tables(sections)

        # Step 3: Detect and render charts
        sections = self._render_charts(sections)

        # Step 4: Generate diagrams (mermaid, D2)
        sections = self._generate_diagrams(sections)

        # Step 5: Format code blocks with language hints
        sections = self._format_code_blocks(sections)

        # Step 6: Add callouts and admonitions
        sections = self._add_callouts(sections)

        # Step 7: Structure heading hierarchy
        sections = self._structure_headings(sections)

        # Step 8: Add YAML frontmatter
        frontmatter = self._generate_frontmatter(metadata)

        # Step 9: Assemble final document
        return MarkdownDocument(
            frontmatter=frontmatter,
            sections=sections,
            metadata=metadata
        )
```

### Table Detection and Formatting

```python
def _format_tables(self, sections: List[Section]) -> List[Section]:
    """
    Detect tables in text and convert to GFM tables.

    Patterns detected:
    - Pipe-separated values
    - ASCII box drawings
    - Aligned columns with spacing
    - Key-value pairs
    """
    for section in sections:
        # Pattern 1: Detect pipe-separated tables
        if self._is_pipe_table(section.text):
            section.text = self._normalize_pipe_table(section.text)

        # Pattern 2: Detect aligned columns
        elif self._is_aligned_table(section.text):
            section.text = self._convert_to_pipe_table(section.text)

        # Pattern 3: Detect ASCII box tables
        elif self._is_box_table(section.text):
            section.text = self._convert_box_to_pipe(section.text)

    return sections

def _normalize_pipe_table(self, text: str) -> str:
    """
    Normalize GFM pipe tables for proper rendering.

    Example input:
    | Header 1 | Header 2|Header 3 |
    |---|---|---|
    |Value 1|Value 2|Value 3|

    Example output:
    | Header 1 | Header 2 | Header 3 |
    |----------|----------|----------|
    | Value 1  | Value 2  | Value 3  |
    """
    lines = text.strip().split('\n')
    rows = [self._parse_pipe_row(line) for line in lines]

    # Calculate column widths
    col_widths = self._calculate_column_widths(rows)

    # Rebuild table with proper alignment
    formatted_rows = []
    for i, row in enumerate(rows):
        if i == 1:  # Header separator
            formatted_rows.append(self._build_separator(col_widths))
        else:
            formatted_rows.append(self._build_row(row, col_widths))

    return '\n'.join(formatted_rows)
```

### Chart Rendering

```python
def _render_charts(self, sections: List[Section]) -> List[Section]:
    """
    Detect chart descriptions and render as ASCII art or mermaid.

    Supported chart types:
    - Bar charts (horizontal/vertical)
    - Timelines
    - Flowcharts
    - Gantt charts
    - Pie charts (text-based)
    """
    for section in sections:
        # Detect chart keywords
        if self._is_chart_description(section.text):
            chart_type = self._detect_chart_type(section.text)

            if chart_type == "bar":
                section.text = self._render_bar_chart(section.text)
            elif chart_type == "timeline":
                section.text = self._render_timeline(section.text)
            elif chart_type == "flowchart":
                section.text = self._render_mermaid_flowchart(section.text)
            elif chart_type == "gantt":
                section.text = self._render_mermaid_gantt(section.text)

    return sections

def _render_bar_chart(self, text: str) -> str:
    """
    Render horizontal bar chart.

    Example input:
    "GDPR Compliance: 85%, EU AI Act: 60%, CCPA: 40%"

    Example output:
    ```
    GDPR Compliance  ████████████████████████████████████ 85%
    EU AI Act        ████████████████████                 60%
    CCPA             ████████████                         40%
    ```
    """
    data = self._parse_chart_data(text)
    max_label_width = max(len(label) for label, _ in data)
    max_value = max(value for _, value in data)

    chart_lines = []
    for label, value in data:
        bar_length = int((value / max_value) * 40)
        bar = "█" * bar_length
        chart_lines.append(f"{label.ljust(max_label_width)}  {bar} {value}%")

    return f"```\n" + "\n".join(chart_lines) + "\n```"
```

### Mermaid Diagram Generation

```python
def _generate_diagrams(self, sections: List[Section]) -> List[Section]:
    """
    Generate mermaid diagrams from text descriptions.

    Supports:
    - Flowcharts
    - Sequence diagrams
    - State diagrams
    - ER diagrams
    - Gantt charts
    """
    for section in sections:
        if self._is_diagram_description(section.text):
            diagram_type = self._detect_diagram_type(section.text)
            mermaid_code = self._convert_to_mermaid(section.text, diagram_type)
            section.text = f"```mermaid\n{mermaid_code}\n```"

    return sections

def _convert_to_mermaid(self, text: str, diagram_type: str) -> str:
    """
    Convert text description to mermaid syntax.

    Example input (flowchart):
    "User submits query → System validates input → RAG retrieval →
     Claude processes → Response formatted → User receives result"

    Example output:
    ```mermaid
    flowchart LR
        A[User submits query] --> B[System validates input]
        B --> C[RAG retrieval]
        C --> D[Claude processes]
        D --> E[Response formatted]
        E --> F[User receives result]
    ```
    """
    if diagram_type == "flowchart":
        return self._build_mermaid_flowchart(text)
    elif diagram_type == "sequence":
        return self._build_mermaid_sequence(text)
    elif diagram_type == "gantt":
        return self._build_mermaid_gantt(text)
    else:
        return text
```

### YAML Frontmatter

```yaml
---
title: "GDPR, EU AI Act & Their Impact on AI Adoption in the EU"
query: "explain link between gdpr, eu ai act and how they are impacting the ai adoption in eu"
model: "eu.anthropic.claude-opus-4-6-v1"
timestamp: "2026-02-11T14:25:33Z"
job_id: "job_20260211_142530_abc123"
user_id: "support@quantamixsolutions.com"
workspace: "default"
confidence_score: 0.92
confidence_tier: "HIGH"
processing_time: 127.5
sections:
  - type: "executive_summary"
    lines: 1-25
  - type: "detailed_analysis"
    lines: 26-180
  - type: "impact_assessment"
    lines: 181-250
  - type: "recommendations"
    lines: 251-300
charts: 3
tables: 8
diagrams: 2
code_blocks: 0
---
```

---

## Frontend Markdown Renderer

### Enhanced React Component

```typescript
// File: crawlq-ui/src/components/chat-eu/MarkdownRenderer/EnterpriseMarkdownRenderer.tsx

import React from 'react';
import ReactMarkdown from 'react-markdown';
import remarkGfm from 'remark-gfm';
import remarkMath from 'remark-math';
import rehypeKatex from 'rehype-katex';
import rehypeMermaid from 'rehype-mermaid';
import rehypeRaw from 'rehype-raw';
import { Prism as SyntaxHighlighter } from 'react-syntax-highlighter';
import { vscDarkPlus } from 'react-syntax-highlighter/dist/esm/styles/prism';
import 'katex/dist/katex.min.css';

interface EnterpriseMarkdownRendererProps {
  content: string;
  metadata?: MarkdownMetadata;
}

export function EnterpriseMarkdownRenderer({
  content,
  metadata
}: EnterpriseMarkdownRendererProps) {
  return (
    <div className="markdown-container">
      {metadata && <MetadataHeader metadata={metadata} />}

      <ReactMarkdown
        remarkPlugins={[remarkGfm, remarkMath]}
        rehypePlugins={[rehypeKatex, rehypeMermaid, rehypeRaw]}
        components={{
          // Code blocks with syntax highlighting
          code: ({ node, inline, className, children, ...props }) => {
            const match = /language-(\w+)/.exec(className || '');
            return !inline && match ? (
              <SyntaxHighlighter
                style={vscDarkPlus}
                language={match[1]}
                PreTag="div"
                {...props}
              >
                {String(children).replace(/\n$/, '')}
              </SyntaxHighlighter>
            ) : (
              <code className={className} {...props}>
                {children}
              </code>
            );
          },

          // Enhanced tables
          table: ({ children }) => (
            <div className="table-wrapper">
              <table className="enterprise-table">
                {children}
              </table>
            </div>
          ),

          // Callout boxes
          blockquote: ({ children }) => {
            const firstChild = React.Children.toArray(children)[0];
            const text = firstChild?.props?.children || '';

            if (text.startsWith('ℹ️')) {
              return <InfoCallout>{children}</InfoCallout>;
            } else if (text.startsWith('⚠️')) {
              return <WarningCallout>{children}</WarningCallout>;
            } else if (text.startsWith('✅')) {
              return <SuccessCallout>{children}</SuccessCallout>;
            }

            return <blockquote>{children}</blockquote>;
          },

          // Checkboxes
          input: ({ type, checked }) => {
            if (type === 'checkbox') {
              return <input type="checkbox" checked={checked} disabled />;
            }
            return <input type={type} />;
          },
        }}
      >
        {content}
      </ReactMarkdown>
    </div>
  );
}
```

### CSS Styling (Tailwind)

```css
/* File: crawlq-ui/src/styles/markdown.css */

.markdown-container {
  @apply max-w-none prose prose-slate dark:prose-invert;
  @apply prose-headings:scroll-mt-20;
  @apply prose-a:text-blue-600 dark:prose-a:text-blue-400;
  @apply prose-code:before:content-none prose-code:after:content-none;
}

.enterprise-table {
  @apply min-w-full divide-y divide-gray-200 dark:divide-gray-700;
  @apply border border-gray-300 dark:border-gray-600 rounded-lg overflow-hidden;
}

.enterprise-table thead {
  @apply bg-gray-50 dark:bg-gray-800;
}

.enterprise-table th {
  @apply px-6 py-3 text-left text-xs font-medium uppercase tracking-wider;
  @apply text-gray-700 dark:text-gray-300;
}

.enterprise-table td {
  @apply px-6 py-4 whitespace-nowrap text-sm;
  @apply border-t border-gray-100 dark:border-gray-800;
}

.table-wrapper {
  @apply overflow-x-auto my-6 rounded-lg shadow-sm;
}

/* Callout boxes */
.callout {
  @apply my-4 p-4 rounded-lg border-l-4;
}

.callout-info {
  @apply bg-blue-50 dark:bg-blue-900/20 border-blue-500;
}

.callout-warning {
  @apply bg-yellow-50 dark:bg-yellow-900/20 border-yellow-500;
}

.callout-success {
  @apply bg-green-50 dark:bg-green-900/20 border-green-500;
}

/* Mermaid diagrams */
.mermaid {
  @apply my-6 p-4 bg-white dark:bg-gray-900 rounded-lg;
}

/* Code blocks */
.code-block-wrapper {
  @apply my-4 rounded-lg overflow-hidden shadow-md;
}
```

---

## Job Polling Hook

```typescript
// File: crawlq-ui/src/queries/chat-eu/useJobPolling.ts

import { useQuery, useQueryClient } from '@tanstack/react-query';
import { useState, useEffect } from 'react';

interface JobStatus {
  job_id: string;
  status: 'pending' | 'rag_retrieval' | 'graph_building' | 'reasoning' | 'formatting' | 'completed' | 'failed';
  progress: number;
  s3_url?: string;
  error?: string;
  estimated_completion?: number;
}

export function useJobPolling(jobId: string | null) {
  const [isPolling, setIsPolling] = useState(false);
  const queryClient = useQueryClient();

  const { data: status, error } = useQuery<JobStatus>({
    queryKey: ['chat-job-status', jobId],
    queryFn: async () => {
      const response = await fetch(
        `${EU_API_BASE}/chat-status/${jobId}`,
        {
          headers: {
            Authorization: `Bearer ${getToken()}`,
          },
        }
      );

      if (!response.ok) {
        throw new Error('Failed to fetch job status');
      }

      return response.json();
    },
    enabled: !!jobId && isPolling,
    refetchInterval: (data) => {
      // Stop polling when completed or failed
      if (data?.status === 'completed' || data?.status === 'failed') {
        setIsPolling(false);
        return false;
      }
      return 2000; // Poll every 2 seconds
    },
  });

  useEffect(() => {
    if (jobId) {
      setIsPolling(true);
    }
  }, [jobId]);

  const fetchResult = async () => {
    if (!status?.s3_url) return null;

    const response = await fetch(status.s3_url);
    const markdown = await response.text();
    return markdown;
  };

  return {
    status,
    error,
    isPolling,
    fetchResult,
    progress: status?.progress || 0,
  };
}
```

### Progress Indicator Component

```typescript
// File: crawlq-ui/src/components/chat-eu/JobProgressIndicator.tsx

interface JobProgressIndicatorProps {
  status: JobStatus;
}

export function JobProgressIndicator({ status }: JobProgressIndicatorProps) {
  const statusConfig = {
    pending: { label: 'Queueing...', icon: '⏳', color: 'gray' },
    rag_retrieval: { label: 'Retrieving context...', icon: '📚', color: 'blue' },
    graph_building: { label: 'Building knowledge graph...', icon: '🕸️', color: 'purple' },
    reasoning: { label: 'AI reasoning...', icon: '🤖', color: 'green' },
    formatting: { label: 'Formatting response...', icon: '✨', color: 'yellow' },
    completed: { label: 'Complete', icon: '✅', color: 'green' },
    failed: { label: 'Failed', icon: '❌', color: 'red' },
  };

  const config = statusConfig[status.status];

  return (
    <div className="flex items-center gap-3 py-3 px-4 bg-gray-50 dark:bg-gray-800 rounded-lg">
      <span className="text-2xl">{config.icon}</span>
      <div className="flex-1">
        <div className="flex items-center justify-between mb-1">
          <span className="text-sm font-medium text-gray-700 dark:text-gray-300">
            {config.label}
          </span>
          <span className="text-xs text-gray-500">
            {status.progress}%
          </span>
        </div>
        <div className="w-full bg-gray-200 dark:bg-gray-700 rounded-full h-2">
          <div
            className={`h-2 rounded-full transition-all duration-500 bg-${config.color}-500`}
            style={{ width: `${status.progress}%` }}
          />
        </div>
        {status.estimated_completion && (
          <span className="text-xs text-gray-400 mt-1">
            ~{status.estimated_completion}s remaining
          </span>
        )}
      </div>
    </div>
  );
}
```

---

## Database Schema

### DynamoDB: eu_chat_jobs

```yaml
Table: eu_chat_jobs
Partition Key: job_id (String)
Sort Key: timestamp (String)

Attributes:
  job_id: String           # UUID v4
  timestamp: String        # ISO 8601
  status: String           # pending | rag_retrieval | graph_building | reasoning | formatting | completed | failed
  progress: Number         # 0-100
  user_id: String          # Cognito sub
  username: String         # Email
  workspace: String        # Workspace name
  question: String         # User query
  answer: String           # Final response (truncated to 400 chars)
  s3_url: String           # S3 URL for full markdown
  s3_key: String           # S3 object key
  model: String            # Model ID
  confidence_score: Number # 0.0-1.0
  error: String            # Error message (if failed)
  started_at: String       # ISO 8601
  completed_at: String     # ISO 8601
  processing_time: Number  # Seconds
  input_tokens: Number
  output_tokens: Number
  ttl: Number              # Unix timestamp (24 hours)

GSI: user-timestamp-index
  Partition Key: user_id
  Sort Key: timestamp
  Purpose: Fetch user's recent jobs

GSI: status-index
  Partition Key: status
  Sort Key: timestamp
  Purpose: Monitor stuck jobs
```

### S3 Bucket: eu-chat-responses

```yaml
Bucket: crawlq-eu-chat-responses
Region: eu-central-1
Encryption: AES256 (SSE-S3)
Versioning: Enabled
CORS: Enabled (read-only for authenticated users)

Object Key Format:
  responses/{year}/{month}/{job_id}.md

Example:
  responses/2026/02/job_20260211_142530_abc123.md

Metadata (x-amz-meta-*):
  user-id: Cognito sub
  username: Email
  model: Model ID
  confidence: 0.92
  tokens-in: 1234
  tokens-out: 5678
  job-id: UUID

Lifecycle Policy:
  - Transition to Intelligent-Tiering: 30 days
  - Archive to Glacier: 90 days
  - Expire: 365 days

Access Control:
  - Lambda: Full access
  - Users: Read-only (presigned URLs via Lambda)
```

---

## Lambda Functions

### 1. EUChatJobQueue

**Purpose:** Accept chat query, create job, push to SQS

```python
# File: crawlq-lambda/SemanticGraphEU/EUChatJobQueue/handler.py

import json
import uuid
import boto3
from datetime import datetime, timedelta

dynamodb = boto3.resource('dynamodb', region_name='eu-central-1')
sqs = boto3.client('sqs', region_name='eu-central-1')

JOBS_TABLE = dynamodb.Table('eu_chat_jobs')
SQS_QUEUE_URL = 'https://sqs.eu-central-1.amazonaws.com/{account}/eu-chat-jobs'

def lambda_handler(event, context):
    body = json.loads(event.get('body', '{}'))

    # Generate job ID
    job_id = f"job_{datetime.utcnow().strftime('%Y%m%d_%H%M%S')}_{uuid.uuid4().hex[:8]}"
    timestamp = datetime.utcnow().isoformat() + 'Z'

    # Extract parameters
    question = body.get('question')
    username = body.get('username')
    workspace = body.get('workspace', 'default')
    user_id = body.get('user_id', username)

    # Create job record
    ttl = int((datetime.utcnow() + timedelta(hours=24)).timestamp())

    JOBS_TABLE.put_item(Item={
        'job_id': job_id,
        'timestamp': timestamp,
        'status': 'pending',
        'progress': 0,
        'user_id': user_id,
        'username': username,
        'workspace': workspace,
        'question': question,
        'answer': '',
        's3_url': '',
        's3_key': '',
        'model': '',
        'confidence_score': 0.0,
        'error': '',
        'started_at': timestamp,
        'ttl': ttl,
    })

    # Push to SQS
    sqs.send_message(
        QueueUrl=SQS_QUEUE_URL,
        MessageBody=json.dumps({
            'job_id': job_id,
            'question': question,
            'username': username,
            'workspace': workspace,
            'user_id': user_id,
            **body,  # Pass all params
        }),
    )

    return {
        'statusCode': 200,
        'body': json.dumps({
            'job_id': job_id,
            'status_url': f'/chat-status/{job_id}',
            'estimated_time': 30,  # seconds
        }),
    }
```

### 2. EUChatJobWorker

**Purpose:** Process job from SQS, invoke Claude, format markdown, save to S3

```python
# File: crawlq-lambda/SemanticGraphEU/EUChatJobWorker/handler.py

import json
import boto3
from datetime import datetime
from markdown_processor import IntelligentMarkdownProcessor

dynamodb = boto3.resource('dynamodb', region_name='eu-central-1')
s3 = boto3.client('s3', region_name='eu-central-1')

JOBS_TABLE = dynamodb.Table('eu_chat_jobs')
S3_BUCKET = 'crawlq-eu-chat-responses'

def update_status(job_id, status, progress):
    JOBS_TABLE.update_item(
        Key={'job_id': job_id, 'timestamp': timestamp},
        UpdateExpression='SET #status = :status, progress = :progress',
        ExpressionAttributeNames={'#status': 'status'},
        ExpressionAttributeValues={':status': status, ':progress': progress},
    )

def lambda_handler(event, context):
    for record in event['Records']:
        message = json.loads(record['body'])
        job_id = message['job_id']

        try:
            # Step 1: RAG retrieval
            update_status(job_id, 'rag_retrieval', 10)
            rag_chunks = fetch_rag_chunks(message['question'], message['username'])

            # Step 2: Graph building
            update_status(job_id, 'graph_building', 30)
            kg_entities = build_kg_entities(rag_chunks)

            # Step 3: Reasoning
            update_status(job_id, 'reasoning', 50)
            claude_response = invoke_claude(
                question=message['question'],
                rag_chunks=rag_chunks,
                kg_entities=kg_entities,
                **message,
            )

            # Step 4: Formatting
            update_status(job_id, 'formatting', 80)
            processor = IntelligentMarkdownProcessor()
            markdown_doc = processor.process(
                claude_response=claude_response['answer'],
                metadata={
                    'job_id': job_id,
                    'model': claude_response['model_used'],
                    'confidence_score': claude_response['confidence_score'],
                    'username': message['username'],
                    'question': message['question'],
                },
            )

            # Step 5: Save to S3
            year_month = datetime.utcnow().strftime('%Y/%m')
            s3_key = f"responses/{year_month}/{job_id}.md"

            s3.put_object(
                Bucket=S3_BUCKET,
                Key=s3_key,
                Body=markdown_doc.to_string(),
                ContentType='text/markdown',
                Metadata={
                    'user-id': message['user_id'],
                    'username': message['username'],
                    'model': claude_response['model_used'],
                    'confidence': str(claude_response['confidence_score']),
                    'job-id': job_id,
                },
            )

            s3_url = s3.generate_presigned_url(
                'get_object',
                Params={'Bucket': S3_BUCKET, 'Key': s3_key},
                ExpiresIn=86400,  # 24 hours
            )

            # Step 6: Mark completed
            JOBS_TABLE.update_item(
                Key={'job_id': job_id, 'timestamp': timestamp},
                UpdateExpression='SET #status = :status, progress = :progress, s3_url = :url, s3_key = :key, answer = :answer, completed_at = :completed',
                ExpressionAttributeNames={'#status': 'status'},
                ExpressionAttributeValues={
                    ':status': 'completed',
                    ':progress': 100,
                    ':url': s3_url,
                    ':key': s3_key,
                    ':answer': claude_response['answer'][:400],  # Truncated
                    ':completed': datetime.utcnow().isoformat() + 'Z',
                },
            )

        except Exception as e:
            # Mark failed
            JOBS_TABLE.update_item(
                Key={'job_id': job_id, 'timestamp': timestamp},
                UpdateExpression='SET #status = :status, #error = :error',
                ExpressionAttributeNames={'#status': 'status', '#error': 'error'},
                ExpressionAttributeValues={':status': 'failed', ':error': str(e)},
            )
```

### 3. EUChatJobStatus

**Purpose:** Poll job status

```python
# File: crawlq-lambda/SemanticGraphEU/EUChatJobStatus/handler.py

import json
import boto3

dynamodb = boto3.resource('dynamodb', region_name='eu-central-1')
JOBS_TABLE = dynamodb.Table('eu_chat_jobs')

def lambda_handler(event, context):
    job_id = event['pathParameters']['job_id']

    # Fetch from DynamoDB
    response = JOBS_TABLE.query(
        KeyConditionExpression='job_id = :job_id',
        ExpressionAttributeValues={':job_id': job_id},
        Limit=1,
    )

    if not response['Items']:
        return {
            'statusCode': 404,
            'body': json.dumps({'error': 'Job not found'}),
        }

    job = response['Items'][0]

    return {
        'statusCode': 200,
        'body': json.dumps({
            'job_id': job['job_id'],
            'status': job['status'],
            'progress': job.get('progress', 0),
            's3_url': job.get('s3_url', ''),
            'error': job.get('error', ''),
            'estimated_completion': estimate_completion(job),
        }),
    }

def estimate_completion(job):
    """Estimate remaining time based on current status."""
    if job['status'] == 'pending': return 30
    if job['status'] == 'rag_retrieval': return 25
    if job['status'] == 'graph_building': return 20
    if job['status'] == 'reasoning': return 15
    if job['status'] == 'formatting': return 5
    return 0
```

---

## Package Dependencies

### Backend (Python)

```txt
# requirements.txt for EUChatJobWorker and EUMarkdownProcessor

boto3==1.35.0
botocore==1.35.0
anthropic==0.40.0
pyyaml==6.0.1
markdown==3.5.2
beautifulsoup4==4.12.3
lxml==5.1.0
pygments==2.17.2  # Syntax highlighting detection
```

### Frontend (NPM)

```json
{
  "dependencies": {
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
}
```

---

## API Gateway Routes

```yaml
API: eu-chat-athena-api (HTTP API)
Region: eu-central-1

Routes:
  POST /chat-async:
    Integration: Lambda (EUChatJobQueue)
    Authorizer: Cognito JWT
    Timeout: 30s (doesn't matter, returns immediately)

  GET /chat-status/{job_id}:
    Integration: Lambda (EUChatJobStatus)
    Authorizer: Cognito JWT
    Timeout: 5s

  GET /chat-result/{job_id}:
    Integration: S3 (presigned URL redirect)
    Authorizer: Cognito JWT
    Timeout: 5s
```

---

## Deployment Plan

### Phase 1: Infrastructure (Day 1)
1. Create DynamoDB table `eu_chat_jobs`
2. Create S3 bucket `crawlq-eu-chat-responses`
3. Create SQS queue `eu-chat-jobs`
4. Update IAM role with S3/SQS/DynamoDB permissions

### Phase 2: Backend (Days 2-3)
1. Build `EUChatJobQueue` Lambda
2. Build `EUChatJobWorker` Lambda
3. Build `EUChatJobStatus` Lambda
4. Build `IntelligentMarkdownProcessor` module
5. Deploy and test end-to-end

### Phase 3: Frontend (Days 4-5)
1. Create `EnterpriseMarkdownRenderer` component
2. Create `JobProgressIndicator` component
3. Create `useJobPolling` hook
4. Update `ChatContainer` to use async flow
5. Install npm dependencies
6. Test rendering with sample markdown

### Phase 4: Integration (Day 6)
1. Update API Gateway routes
2. Deploy all Lambdas
3. Update frontend endpoints
4. Deploy to Amplify
5. End-to-end testing

### Phase 5: Polish (Day 7)
1. Error handling edge cases
2. Loading states
3. Retry mechanisms
4. Monitoring and alerts
5. Documentation

---

## Success Metrics

- ✅ Zero 503 timeout errors
- ✅ Support queries up to 15 minutes processing time
- ✅ Tables render perfectly (GFM format)
- ✅ Charts render as ASCII art or mermaid
- ✅ Code blocks have syntax highlighting
- ✅ Every response saved as markdown file in S3
- ✅ Users see progress indicators (5 stages)
- ✅ Rendering quality matches Claude Code
- ✅ Markdown files include YAML frontmatter with metadata

---

## Related ADRs

- ADR-010: API Gateway HTTP API + Cognito JWT Authorizer
- ADR-011: API Gateway Timeout Handling Strategy (Tier 1-3)
- ADR-008: Claude Opus 4.6 as Primary EU Bedrock Model

---

## References

- [AWS Lambda with SQS](https://docs.aws.amazon.com/lambda/latest/dg/with-sqs.html)
- [DynamoDB TTL](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/TTL.html)
- [S3 Presigned URLs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/PresignedUrlUploadObject.html)
- [GitHub Flavored Markdown](https://github.github.com/gfm/)
- [Mermaid Diagrams](https://mermaid.js.org/)
- [react-markdown](https://github.com/remarkjs/react-markdown)
- [remark-gfm](https://github.com/remarkjs/remark-gfm)
