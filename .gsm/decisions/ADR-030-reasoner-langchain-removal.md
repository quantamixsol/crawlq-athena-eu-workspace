# ADR-030: EU Reasoner - LangChain Dependency Removed (Rule-Based Python)

**Date:** 2026-02-13 | **Status:** ACCEPTED

## Context

The `eu_reasoner` Lambda function was inherited from the US codebase with LangChain dependencies (`langchain==0.2.16`, `langchain-anthropic==0.1.23`, `anthropic==0.34.2`). Initial deployment attempts using pip-based packaging encountered dependency conflicts on Windows environments.

Investigation revealed that while LangChain was imported and instantiated, it was **never actually used** in the code. The entire reasoning logic is implemented as rule-based Python:

### Code Analysis

**LangChain Import (Unused):**
```python
# helpers.py line 15
from langchain_anthropic import ChatAnthropic

# helpers.py line 171-175
class MultiAgentOrchestrator:
    def __init__(self):
        self.llm = ChatAnthropic(
            model=ReasonerConfig.MODEL,
            temperature=ReasonerConfig.TEMPERATURE,
            anthropic_api_key=ReasonerConfig.ANTHROPIC_API_KEY,
        ) if ReasonerConfig.ANTHROPIC_API_KEY else None
```

**LangChain Usage (Zero Instances):**
```bash
$ grep -r "self\.llm\." SemanticGraphEU/EUReasoner/
# No matches found
```

The `self.llm` instance is created but **never called** anywhere in the codebase.

### Actual Implementation (Rule-Based)

All reasoning is implemented as deterministic Python logic:

```python
def _analyze_documents(self, documents, insights, trace):
    """Rule-based document analysis"""
    analyses = []
    for doc in documents:
        doc_insights = [i for i in insights if i['documentId'] == doc['documentId']]
        severity_counts = {'HIGH': 0, 'MEDIUM': 0, 'LOW': 0}
        for insight in doc_insights:
            severity_counts[insight.get('severity', 'LOW')] += 1

        analyses.append({
            'document_id': doc['documentId'],
            'filename': doc.get('filename', 'Unknown'),
            'high_severity_count': severity_counts['HIGH'],
            'medium_severity_count': severity_counts['MEDIUM'],
            'low_severity_count': severity_counts['LOW'],
            'total_insights': len(doc_insights)
        })
    return analyses

def _find_cross_document_relationships(self, insights, trace):
    """Rule-based relationship detection"""
    relationships = []
    insight_dict = {}
    for insight in insights:
        key = insight.get('category', 'general')
        insight_dict.setdefault(key, []).append(insight)

    for category, related_insights in insight_dict.items():
        if len(related_insights) > 1:
            relationships.append({
                'category': category,
                'document_count': len(set(i['documentId'] for i in related_insights)),
                'insight_count': len(related_insights)
            })
    return relationships

def _calculate_confidence(self, doc_count, insight_count, rel_count):
    """Rule-based confidence scoring"""
    base = 0.5
    doc_factor = min(doc_count * 0.1, 0.2)
    insight_factor = min(insight_count * 0.01, 0.2)
    rel_factor = min(rel_count * 0.05, 0.1)
    return min(base + doc_factor + insight_factor + rel_factor, 1.0)
```

No LLM calls, no prompt templates, no chain orchestration — just pure Python rules.

## Decision

**Remove LangChain dependencies entirely.** Deploy as a simple ZIP package with minimal dependencies:

### Before (requirements.txt with LangChain):
```txt
boto3==1.34.0
tenacity==8.2.3
PyJWT==2.8.0
langchain==0.2.16
langchain-anthropic==0.1.23
langchain-core==0.2.38
anthropic==0.34.2
```

### After (requirements.txt without LangChain):
```txt
boto3==1.34.0
tenacity==8.2.3
PyJWT==2.8.0
```

### Code Changes:
```python
# Remove unused imports
- from langchain_anthropic import ChatAnthropic
- from langchain.tools import tool

# Remove unused class attribute
class MultiAgentOrchestrator:
    def __init__(self):
-       self.llm = ChatAnthropic(...)
        pass  # No LLM needed - all logic is rule-based
```

## Consequences

### Positive

1. **Zero Dependency Conflicts**: No langchain version mismatches, no anthropic SDK conflicts, no pip --user flag issues on Windows
2. **Smaller Package Size**: 0.03 MB (30 KB) vs projected 15+ MB with LangChain dependencies
3. **Faster Cold Starts**: Simple Python imports vs LangChain's complex dependency graph
4. **ZIP Deployment**: No Docker needed, autonomous boto3 deployment from Windows works perfectly
5. **Easier Maintenance**: Pure Python rule-based logic is easier to debug and modify than LangChain abstractions
6. **No External API Dependencies**: No Anthropic API calls → no quota concerns, no rate limits, no API key rotation
7. **Deterministic Results**: Same inputs always produce same outputs (no LLM variability)

### Negative

1. **No LLM Reasoning**: If future requirements need actual Claude-based reasoning, we'd need to add it back (or use Bedrock directly via boto3)
2. **Limited Intelligence**: Rule-based logic can't handle nuanced semantic reasoning tasks that an LLM could

### Mitigation

If LLM-based reasoning becomes required in the future:
- Use `boto3` Bedrock client directly (already available, no new dependencies)
- Example: `bedrock_runtime.invoke_model(modelId='anthropic.claude-opus-4-6', body=json.dumps(...))`
- Avoids LangChain dependency entirely while gaining LLM capabilities

## Deployment Details

**Lambda Version:** 3
**Package Size:** 0.03 MB (30 KB)
**Deployment Method:** boto3 ZIP upload (no Docker)
**Deployment Script:** `deploy_reasoner_fix.py`

```python
import boto3
import zipfile
import os

lambda_client = boto3.client('lambda', region_name='eu-central-1')

# Package without LangChain dependencies
with zipfile.ZipFile('reasoner_package.zip', 'w') as z:
    z.write('handler.py')
    z.write('helpers.py')
    # No site-packages needed - only stdlib + boto3 (already in Lambda runtime)

# Deploy
lambda_client.update_function_code(
    FunctionName='eu_reasoner',
    ZipFile=open('reasoner_package.zip', 'rb').read()
)
```

**Deployed:** 2026-02-13T02:45:00Z
**Lambda ARN:** `arn:aws:lambda:eu-central-1:680341090470:function:eu_reasoner:3`

## Related ADRs

- **ADR-004**: ZIP-Based Lambda Deployment from Windows
- **ADR-014**: Use boto3 for Lambda Deployment Over AWS CLI
- **ADR-021**: Standard Lambda Deployment Tool (boto3 + ZIP + Function URLs)

## Lessons Learned

1. **Always verify dependency usage** before attempting complex Docker builds. grep for actual usage, not just imports.
2. **LangChain is not always necessary**. For rule-based logic, pure Python is simpler, faster, and more maintainable.
3. **Windows pip --user flag conflicts** can block deployment. ZIP packaging with only runtime dependencies avoids this entirely.
4. **Inherited code may contain unused dependencies** from previous implementations. Refactoring opportunities exist.
