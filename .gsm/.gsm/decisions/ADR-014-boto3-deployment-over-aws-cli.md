# ADR-014: Use boto3 for Lambda Deployment Over AWS CLI

**Date:** 2026-02-11
**Status:** ACCEPTED
**Decision Makers:** Claude Code Agent
**Context:** Tier 3 async job queue Lambda deployment

---

## Context

During the deployment of Tier 3 Lambda functions (EUChatJobQueue, EUChatJobStatus, EUChatJobWorker), we encountered issues where AWS CLI was not available or properly configured in the development environment. This required a decision on the deployment approach.

### Initial Approach
- Created bash scripts using AWS CLI commands
- Attempted Docker-based container deployments
- Failed due to:
  1. AWS CLI not in PATH (`aws: command not found`)
  2. Docker not installed
  3. Attempted to direct user to manual execution

### Problem Statement
The agent must be able to deploy AWS Lambda functions programmatically without relying on external CLI tools that may or may not be installed on the user's system.

---

## Decision

**We will use boto3 (AWS SDK for Python) directly for all Lambda deployments instead of relying on AWS CLI.**

### Rationale

1. **boto3 is Already Available**
   - boto3 version 1.42.45 was already installed in the Python environment
   - Python 3.10.11 was available and working
   - No additional installations needed

2. **More Reliable**
   - boto3 provides programmatic access to AWS APIs
   - No dependency on shell environment or PATH configuration
   - No dependency on Docker installation

3. **Better Error Handling**
   - Python exceptions are easier to catch and handle
   - Can implement retries, waits, and validation programmatically
   - Better logging and debugging capabilities

4. **Cross-Platform Compatibility**
   - boto3 works consistently across Windows, Linux, and macOS
   - No shell script compatibility issues
   - No need for Git Bash, WSL, or specific terminal emulators

5. **Direct Control**
   - Can query AWS state before making changes
   - Can validate configurations programmatically
   - Can implement custom deployment logic (e.g., skip if already deployed)

---

## Implementation

### Created: `deploy_tier3_zip.py`
A Python script using boto3 that:
- Creates ZIP deployment packages for Lambda functions
- Installs dependencies using `pip install -t`
- Deploys/updates Lambda functions using `lambda_client.create_function()` / `update_function_code()`
- Creates Lambda Function URLs with CORS configuration
- Configures SQS event source mappings
- Handles permissions (`add_permission()` for public access)
- Waits for functions to be active using `get_waiter()`

### Key boto3 Operations Used
```python
# IAM
iam.get_role(RoleName='...')

# Lambda
lambda_client.create_function(...)
lambda_client.update_function_code(...)
lambda_client.update_function_configuration(...)
lambda_client.create_function_url_config(...)
lambda_client.add_permission(...)
lambda_client.create_event_source_mapping(...)

# SQS
sqs.get_queue_url(QueueName='...')

# Waiters
lambda_client.get_waiter('function_active').wait(...)
lambda_client.get_waiter('function_updated').wait(...)
```

### Advantages Over AWS CLI
| Aspect | AWS CLI | boto3 |
|--------|---------|-------|
| Availability | May not be installed | Already available with Python |
| Error Handling | Shell exit codes | Python exceptions |
| Validation | Manual checks | Programmatic queries |
| Cross-platform | Shell script issues | Works everywhere |
| Debugging | `set -x`, logs | Python debugger, print statements |
| Idempotency | Manual if checks | Easy try/except with `ResourceConflictException` |

---

## Consequences

### Positive
1. **Immediate Deployment Success**
   - All 3 Lambda functions deployed successfully using boto3
   - Function URLs created and configured
   - SQS trigger configured
   - Completed in < 60 seconds

2. **No User Intervention Required**
   - Agent can deploy autonomously
   - No need to ask user to run scripts manually
   - No need to troubleshoot CLI installation issues

3. **Reproducible**
   - Same script works in any environment with Python + boto3
   - No dependency on shell environment
   - Easy to version control and test

4. **Easier Maintenance**
   - Python code is easier to modify than bash scripts
   - Better IDE support (autocomplete, type hints)
   - Can use Python testing frameworks

### Negative
1. **More Code**
   - ~300 lines of Python vs. ~100 lines of bash
   - Need to handle more edge cases programmatically

2. **Requires Python**
   - If Python isn't available, script won't work
   - But Python is more commonly available than AWS CLI configured correctly

3. **Learning Curve**
   - Developers need to know boto3 API
   - But boto3 documentation is excellent

### Trade-offs
- **Verbosity vs. Reliability**: More code but higher success rate
- **Flexibility vs. Simplicity**: Can handle complex scenarios but requires more Python knowledge

---

## Lessons Learned

1. **Never Assume CLI Availability**
   - Even if AWS CLI was used in previous sessions, it may not be available in current environment
   - Different terminals (Git Bash, PowerShell, WSL) have different PATH configurations

2. **SDK Over CLI for Automation**
   - When building automation, prefer SDK (boto3, AWS SDK for JavaScript, etc.) over CLI
   - SDKs provide better error handling and programmatic control

3. **Validate Environment Early**
   - Check for required tools (boto3, pip) before attempting deployment
   - Provide clear error messages if prerequisites are missing

4. **Document Both Approaches**
   - Keep bash scripts for manual deployment
   - Use boto3 for automated deployment
   - Document when to use each approach

---

## Related Decisions

- **ADR-012**: Tier 3 Async Job Queue Architecture (defines the Lambda functions being deployed)
- **ADR-011**: EU-Only Segmentation Architecture (defines deployment region constraints)
- **ADR-013**: US Region Non-Interference Policy (ensures no accidental US deployments)

---

## Action Items

- [x] Create `deploy_tier3_zip.py` using boto3
- [x] Deploy all 3 Tier 3 Lambda functions
- [x] Create Function URLs for queue and status functions
- [x] Configure SQS trigger for worker function
- [x] Fix Lambda permissions (add `lambda:InvokeFunction` permission)
- [x] Fix Lambda handlers (query string parameters for Function URLs)
- [x] Fix DynamoDB composite key queries
- [x] Fix Decimal serialization for DynamoDB/SQS
- [ ] Update deployment documentation to recommend boto3 over AWS CLI
- [ ] Create boto3 deployment templates for other services (API Gateway, S3, etc.)

---

## Future Considerations

1. **Extend to Other Services**
   - Use boto3 for DynamoDB provisioning
   - Use boto3 for S3 bucket creation
   - Use boto3 for API Gateway configuration

2. **Create Reusable Modules**
   - `lambda_deployer.py` - reusable Lambda deployment module
   - `aws_utils.py` - common AWS operations
   - `deployment_config.py` - configuration management

3. **Add Rollback Capability**
   - Store previous Lambda versions
   - Implement rollback on deployment failure
   - Add deployment history tracking

4. **Monitoring and Alerts**
   - Add CloudWatch log tailing during deployment
   - Implement health checks after deployment
   - Add Slack/email notifications on deployment success/failure

---

## Conclusion

Using boto3 for Lambda deployment proved to be more reliable and maintainable than AWS CLI-based approaches. This decision enables autonomous agent deployment without requiring user intervention for CLI setup. All future AWS resource deployments should prefer boto3 over AWS CLI unless there's a specific reason to use CLI (e.g., interactive debugging, one-off operations).

**Recommendation:** Always use boto3 for programmatic AWS deployments. Never ask for AWS CLI when boto3 is available.
