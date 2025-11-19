# Redis to DynamoDB Migration Summary

**Date:** November 17, 2025  
**Update Type:** Infrastructure Change  
**Scope:** Session caching layer replacement

-----

## Overview

Replaced Redis with AWS DynamoDB for session data caching in the Lizzy Agentic Commerce CRM system. This change provides better integration with AWS infrastructure, simplified operational management, and cost-effective scaling.

-----

## Changes Made

### 1. System Architecture

**Previous:**
- Redis - Session data caching (customer profile, equipment, parts, transactions)

**Updated:**
- **DynamoDB** - Session data caching (customer profile, equipment, parts, transactions)

### 2. Session Management

**Added TTL Configuration:**
- DynamoDB automatically expires sessions after 15 minutes of inactivity
- TTL attribute set to current timestamp + 900 seconds
- No manual cleanup required

### 3. Backend Implementation

**DynamoDB Table Structure:**

```json
{
  "phone": "+12065550100",                    // Partition Key
  "customerId": "CUST123",
  "currentNode": "MAIN_MENU",
  "cachedData": {
    "customerProfile": { /* ... */ },
    "profileCacheTime": "2025-11-11T10:30:00Z",
    "transactions": { /* ... */ },
    "transactionsCacheTime": "2025-11-11T10:35:00Z"
  },
  "sessionData": {
    "selectedEquipmentId": "EQ123",
    "selectedEquipmentName": "Honda HRX217",
    "selectedEquipmentSN": "MZCG-1234567",
    "filterUnitType": "Mowers",
    "filterModel": "HRX217",
    "problemDescription": "Engine won't start",
    "partsOrderRequested": false
  },
  "lastActivity": "2025-11-11T10:40:00Z",
  "ttl": 1731325500                           // TTL attribute
}
```

**Key Configuration:**
- **Table Name:** `LizzySMSSessions`
- **Partition Key:** `phone` (String) - E.164 format
- **Capacity Mode:** On-Demand (recommended)
- **TTL Enabled:** Yes, on `ttl` attribute
- **Indexes:** None required for MVP

-----

## Benefits of DynamoDB Over Redis

### 1. Operational Simplicity
- **No server management** - Fully serverless
- **No capacity planning** - On-Demand mode auto-scales
- **No patching/updates** - AWS manages infrastructure
- **Built-in TTL** - Automatic session expiration without manual cleanup

### 2. Cost Efficiency
- **Pay per request** - No idle server costs
- **No minimum commitment** - Start small, scale as needed
- **Predictable pricing** - Easy to estimate based on traffic

### 3. AWS Integration
- **Native AWS service** - Better integration if using other AWS services
- **IAM-based access control** - Seamless security management
- **VPC endpoints** - Private connectivity without internet gateway
- **CloudWatch integration** - Built-in monitoring and alarms

### 4. Reliability
- **99.99% availability SLA** - High durability
- **Multi-AZ replication** - Automatic failover
- **Point-in-time recovery** - Backup and restore capability
- **No single point of failure** - Distributed architecture

### 5. Scalability
- **Unlimited throughput** - On-Demand mode handles traffic spikes
- **Global tables** - Multi-region support if needed
- **Low latency** - Single-digit millisecond response times

-----

## Cost Comparison

### Redis (AWS ElastiCache)
**Minimum Configuration:**
- cache.t3.micro (0.5GB): ~$12/month
- cache.t3.small (1.37GB): ~$25/month
- **Always running** - Pay even with zero traffic

**For Lizzy Project (estimated 1-2GB):**
- cache.t3.small: ~$25-30/month
- Standard tier (with replication): ~$50-60/month

### DynamoDB (On-Demand)
**Pricing:**
- Write: $1.25 per million writes
- Read: $0.25 per million reads
- Storage: $0.25 per GB-month

**For Lizzy Project (estimated usage):**

Assumptions:
- 1,000 SMS sessions/month
- Average 5 operations per session (1 write on start, 3 reads/writes during, 1 final write)
- Average session size: 50KB
- Storage: ~0.5GB (concurrent sessions)

**Monthly Cost Calculation:**
- Writes: 3,000 writes × $1.25/million = $0.004
- Reads: 2,000 reads × $0.25/million = $0.0005
- Storage: 0.5GB × $0.25 = $0.125

**Total: ~$0.13/month** (essentially free for low volume)

Even at **10,000 sessions/month**: ~$1.30/month

**Break-even point:** ~200,000 sessions/month to match Redis cost

-----

## Implementation Requirements

### 1. AWS SDK Installation

```bash
npm install @aws-sdk/client-dynamodb @aws-sdk/lib-dynamodb
```

### 2. DynamoDB Table Creation

**Using AWS CLI:**

```bash
aws dynamodb create-table \
  --table-name LizzySMSSessions \
  --attribute-definitions AttributeName=phone,AttributeType=S \
  --key-schema AttributeName=phone,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST \
  --tags Key=Project,Value=LizzyAgenticCommerce
```

**Enable TTL:**

```bash
aws dynamodb update-time-to-live \
  --table-name LizzySMSSessions \
  --time-to-live-specification "Enabled=true, AttributeName=ttl"
```

### 3. IAM Permissions

Required permissions for the workflow engine:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:PutItem",
        "dynamodb:UpdateItem",
        "dynamodb:DeleteItem"
      ],
      "Resource": "arn:aws:dynamodb:*:*:table/LizzySMSSessions"
    }
  ]
}
```

### 4. Code Updates

**Session Operations:**

```javascript
const { DynamoDBClient } = require("@aws-sdk/client-dynamodb");
const { DynamoDBDocumentClient, GetCommand, PutCommand } = require("@aws-sdk/lib-dynamodb");

const client = new DynamoDBClient({ region: "us-east-1" });
const docClient = DynamoDBDocumentClient.from(client);

// Get session
async function getSession(phone) {
  const command = new GetCommand({
    TableName: "LizzySMSSessions",
    Key: { phone }
  });
  const response = await docClient.send(command);
  return response.Item;
}

// Save session
async function saveSession(session) {
  const ttl = Math.floor(Date.now() / 1000) + 900; // 15 minutes
  const command = new PutCommand({
    TableName: "LizzySMSSessions",
    Item: {
      ...session,
      lastActivity: new Date().toISOString(),
      ttl
    }
  });
  await docClient.send(command);
}
```

-----

## Migration Path (If Moving from Existing Redis)

### Phase 1: Setup
1. Create DynamoDB table
2. Enable TTL
3. Configure IAM permissions
4. Install AWS SDK

### Phase 2: Parallel Operation
1. Write to both Redis and DynamoDB
2. Read from Redis (fallback to DynamoDB)
3. Monitor DynamoDB performance
4. Validate data consistency

### Phase 3: Cutover
1. Switch read path to DynamoDB
2. Keep Redis as backup for 1 week
3. Monitor for issues

### Phase 4: Cleanup
1. Remove Redis dependencies from code
2. Decommission Redis instance
3. Update documentation

-----

## Testing Considerations

### Unit Tests
- Session creation/retrieval
- TTL expiration behavior
- Cache refresh logic
- Data filtering operations

### Integration Tests
- End-to-end workflow with DynamoDB
- Session timeout scenarios
- Concurrent session handling
- Error handling (DynamoDB unavailable)

### Load Tests
- Simulate expected traffic patterns
- Test auto-scaling behavior
- Verify latency requirements
- Monitor costs during testing

-----

## Monitoring & Observability

### CloudWatch Metrics to Monitor
- **ConsumedReadCapacityUnits** - Read usage
- **ConsumedWriteCapacityUnits** - Write usage
- **UserErrors** - Application errors
- **SystemErrors** - DynamoDB errors
- **ThrottledRequests** - Rate limiting events

### Recommended Alarms
- SystemErrors > 0 (5 min period)
- UserErrors > 10 (5 min period)
- ThrottledRequests > 0 (if using provisioned capacity)

### Cost Tracking
- Enable Cost Explorer
- Set budget alerts
- Tag resources for cost allocation

-----

## Rollback Plan

If issues arise with DynamoDB:

1. **Immediate:** Keep PostgreSQL as backup session store
2. **Short-term:** Revert to Redis if already deployed
3. **Data Recovery:** Sessions expire naturally, no migration needed

-----

## Next Steps

1. ✅ Update requirements document
2. ✅ Document DynamoDB table structure
3. ⏳ Create DynamoDB table in AWS account
4. ⏳ Update workflow engine code to use DynamoDB SDK
5. ⏳ Test session operations locally
6. ⏳ Deploy and monitor in staging environment
7. ⏳ Conduct load testing
8. ⏳ Deploy to production

-----

## Questions for Implementation

1. Which AWS region should host the DynamoDB table?
2. Do we need multi-region deployment for disaster recovery?
3. Should we enable point-in-time recovery for backups?
4. What CloudWatch alarms should be configured?
5. Should we use VPC endpoints for private connectivity?
6. What's the tagging strategy for cost allocation?

-----

## Summary

The migration from Redis to DynamoDB provides:
- **99% cost reduction** for expected traffic volumes
- **Zero operational overhead** - fully managed
- **Better AWS integration** - native service
- **Automatic TTL management** - no manual cleanup
- **Improved reliability** - 99.99% SLA
- **Unlimited scaling** - handles traffic spikes

This change aligns with modern serverless architecture patterns and significantly reduces infrastructure costs while improving reliability and maintainability.
