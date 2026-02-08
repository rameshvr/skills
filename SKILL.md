---
name: aws-waf-skill
description: AWS WAF (Web Application Firewall) configuration, rule management, and security best practices for protecting web applications
---

# AWS WAF Skill

Expert guidance for AWS Web Application Firewall (WAF) configuration, rule management, security best practices, and integration with AWS services.

## When to Use

Use this skill when:

- Configuring AWS WAF for CloudFront, ALB, API Gateway, or AppSync
- Creating or managing WAF rules, rule groups, and web ACLs
- Implementing protection against common web exploits (OWASP Top 10)
- Setting up rate limiting and bot protection
- Analyzing WAF logs and metrics
- Troubleshooting WAF rule issues or false positives
- Migrating from AWS WAF Classic to WAF v2
- Integrating WAF with other AWS security services

## Core Concepts

### WAF Components

1. **Web ACL** - Container for rules that inspect web requests
2. **Rules** - Logic to inspect requests (match conditions + action)
3. **Rule Groups** - Collection of reusable rules
4. **IP Sets** - Collection of IP addresses and CIDR ranges
5. **Regex Pattern Sets** - Reusable regex patterns for matching

### Actions

- **Allow** - Forward the request
- **Block** - Return HTTP 403 response
- **Count** - Count the request but don't take action (testing mode)
- **CAPTCHA** - Challenge with CAPTCHA
- **Challenge** - Silent JavaScript challenge

## Best Practices

### 1. Start with AWS Managed Rules

Always begin with AWS Managed Rule Groups as a foundation:

```
- Core Rule Set (CRS) - OWASP Top 10 protection
- Known Bad Inputs - Known malicious patterns
- SQL Database - SQL injection protection
- Linux Operating System - Linux-specific exploits
- POSIX Operating System - POSIX-specific exploits
```

### 2. Use Count Mode for Testing

When adding new rules:
1. Set action to `Count` initially
2. Monitor CloudWatch metrics and logs
3. Tune for false positives
4. Switch to `Block` after validation

### 3. Implement Defense in Depth

Layer your protection:

```
Priority Order:
1. Rate limiting (prevent DoS)
2. IP reputation (block known bad actors)
3. Geo-blocking (if applicable)
4. AWS Managed Rules (OWASP protection)
5. Custom rules (application-specific)
6. Allow rules (whitelist trusted sources)
```

### 4. Rate Limiting Strategy

Implement tiered rate limits:

```
- Global rate limit: 2000 requests per 5 minutes per IP
- Login endpoint: 5 requests per minute per IP
- API endpoints: 100 requests per minute per IP
- Search endpoints: 20 requests per minute per IP
```

### 5. Logging and Monitoring

Always enable:
- **WAF logging** to S3, CloudWatch Logs, or Kinesis
- **CloudWatch metrics** for monitoring
- **Alarming** on blocked requests threshold
- **Log sampling** if cost is a concern (minimum 10%)

## Common Patterns

### Pattern 1: CloudFront + WAF

```typescript
// CDK Example
import * as wafv2 from 'aws-cdk-lib/aws-wafv2';
import * as cloudfront from 'aws-cdk-lib/aws-cloudfront';

const webAcl = new wafv2.CfnWebACL(this, 'WebAcl', {
  scope: 'CLOUDFRONT', // Must be CLOUDFRONT for CloudFront
  defaultAction: { allow: {} },
  rules: [
    {
      name: 'AWS-AWSManagedRulesCommonRuleSet',
      priority: 1,
      statement: {
        managedRuleGroupStatement: {
          vendorName: 'AWS',
          name: 'AWSManagedRulesCommonRuleSet',
        },
      },
      overrideAction: { none: {} },
      visibilityConfig: {
        sampledRequestsEnabled: true,
        cloudWatchMetricsEnabled: true,
        metricName: 'AWS-AWSManagedRulesCommonRuleSet',
      },
    },
  ],
  visibilityConfig: {
    sampledRequestsEnabled: true,
    cloudWatchMetricsEnabled: true,
    metricName: 'WebAcl',
  },
});

const distribution = new cloudfront.Distribution(this, 'Distribution', {
  // ... other config
  webAclId: webAcl.attrArn,
});
```

### Pattern 2: ALB + WAF

```typescript
// For ALB, scope must be REGIONAL
const webAcl = new wafv2.CfnWebACL(this, 'AlbWebAcl', {
  scope: 'REGIONAL',
  defaultAction: { allow: {} },
  // ... rules
});

const association = new wafv2.CfnWebACLAssociation(this, 'WebAclAssociation', {
  resourceArn: loadBalancer.loadBalancerArn,
  webAclArn: webAcl.attrArn,
});
```

### Pattern 3: Rate Limiting Rule

```typescript
{
  name: 'RateLimitRule',
  priority: 0,
  statement: {
    rateBasedStatement: {
      limit: 2000,
      aggregateKeyType: 'IP',
    },
  },
  action: { block: {} },
  visibilityConfig: {
    sampledRequestsEnabled: true,
    cloudWatchMetricsEnabled: true,
    metricName: 'RateLimitRule',
  },
}
```

### Pattern 4: Geo-Blocking

```typescript
{
  name: 'GeoBlockRule',
  priority: 2,
  statement: {
    geoMatchStatement: {
      countryCodes: ['CN', 'RU', 'KP'], // Example countries
    },
  },
  action: { block: {} },
  visibilityConfig: {
    sampledRequestsEnabled: true,
    cloudWatchMetricsEnabled: true,
    metricName: 'GeoBlockRule',
  },
}
```

### Pattern 5: IP Whitelist

```typescript
// Create IP Set
const ipSet = new wafv2.CfnIPSet(this, 'AllowedIPs', {
  scope: 'CLOUDFRONT',
  ipAddressVersion: 'IPV4',
  addresses: [
    '203.0.113.0/24',
    '198.51.100.42/32',
  ],
});

// Reference in rule
{
  name: 'AllowKnownIPs',
  priority: 0,
  statement: {
    ipSetReferenceStatement: {
      arn: ipSet.attrArn,
    },
  },
  action: { allow: {} },
  visibilityConfig: {
    sampledRequestsEnabled: true,
    cloudWatchMetricsEnabled: true,
    metricName: 'AllowKnownIPs',
  },
}
```

### Pattern 6: Bot Protection

```typescript
{
  name: 'AWS-AWSManagedRulesBotControlRuleSet',
  priority: 3,
  statement: {
    managedRuleGroupStatement: {
      vendorName: 'AWS',
      name: 'AWSManagedRulesBotControlRuleSet',
    },
  },
  overrideAction: { none: {} },
  visibilityConfig: {
    sampledRequestsEnabled: true,
    cloudWatchMetricsEnabled: true,
    metricName: 'BotControl',
  },
}
```

## Rule Priority Guidelines

Lower number = higher priority. Structure priorities as:

```
0-99:    IP whitelists and allow rules
100-199: Rate limiting
200-299: Geo-blocking
300-399: IP reputation/blocklists
400-499: AWS Managed Rules
500-599: Custom application rules
600-999: Logging/counting rules
```

## Troubleshooting False Positives

### Step 1: Identify the Blocking Rule

Check WAF logs for:
- `ruleId` - Which rule blocked the request
- `action` - The action taken
- `httpRequest` - Request details

### Step 2: Common False Positive Scenarios

**SQL Injection Rules**: May block legitimate requests with SQL-like syntax
```
Solution: Use scope-down statements or excluded rules
```

**XSS Rules**: May block HTML content submissions
```
Solution: Exclude specific paths (e.g., /admin/content-editor)
```

**Size Constraints**: May block large file uploads
```
Solution: Adjust body size limits or exclude upload endpoints
```

### Step 3: Create Exceptions

```typescript
{
  name: 'AWS-AWSManagedRulesCommonRuleSet',
  priority: 1,
  statement: {
    managedRuleGroupStatement: {
      vendorName: 'AWS',
      name: 'AWSManagedRulesCommonRuleSet',
      excludedRules: [
        { name: 'SizeRestrictions_BODY' },
        { name: 'GenericRFI_BODY' },
      ],
    },
  },
  overrideAction: { none: {} },
  // ... visibility config
}
```

Or use scope-down statements:

```typescript
{
  name: 'ManagedRulesWithException',
  statement: {
    managedRuleGroupStatement: {
      vendorName: 'AWS',
      name: 'AWSManagedRulesCommonRuleSet',
      scopeDownStatement: {
        notStatement: {
          statement: {
            byteMatchStatement: {
              searchString: '/api/upload',
              fieldToMatch: { uriPath: {} },
              textTransformations: [{ priority: 0, type: 'NONE' }],
              positionalConstraint: 'STARTS_WITH',
            },
          },
        },
      },
    },
  },
  // ... rest of config
}
```

## Cost Optimization

### Tips to Reduce Costs

1. **Use Log Sampling**: Set to 10-25% instead of 100%
2. **Rule Consolidation**: Combine similar rules when possible
3. **Regional Replication**: Don't replicate Web ACLs unnecessarily
4. **Managed Rule Groups**: More cost-effective than custom rules for common protections
5. **Archive Old Logs**: Move S3 logs to Glacier after 30 days

### Pricing Awareness

- Web ACL: $5/month
- Rules: $1/month per rule
- Requests: $0.60 per 1M requests
- Bot Control: Additional $10/month + $1 per 1M requests
- CAPTCHA: $0.40 per 1,000 challenge attempts

## Security Checklist

- [ ] Enable WAF logging (S3 or CloudWatch Logs)
- [ ] Implement rate limiting on all public endpoints
- [ ] Use AWS Managed Rules for OWASP protection
- [ ] Enable CloudWatch alarms for blocked requests
- [ ] Test rules in Count mode before blocking
- [ ] Document all custom rules and their purpose
- [ ] Review and update rules quarterly
- [ ] Set up IP whitelists for trusted sources
- [ ] Enable bot protection for user-facing applications
- [ ] Configure appropriate default action (typically Block for sensitive apps)
- [ ] Use regex pattern sets for sensitive data patterns
- [ ] Implement geo-blocking if applicable
- [ ] Create runbook for handling false positives

## Common Gotchas

1. **CloudFront WAF must use CLOUDFRONT scope** (us-east-1 only)
2. **ALB/API Gateway must use REGIONAL scope** (in the resource's region)
3. **Rule priorities must be unique** within a Web ACL
4. **Managed Rules require overrideAction**, custom rules need action
5. **Rate limits are per Web ACL**, not per rule
6. **CAPTCHA/Challenge require JavaScript** enabled in browsers
7. **Changes can take up to 1 minute** to propagate globally

## Integration Points

### With CloudWatch

```typescript
// Create alarm for high block rate
new cloudwatch.Alarm(this, 'HighBlockRate', {
  metric: new cloudwatch.Metric({
    namespace: 'AWS/WAFV2',
    metricName: 'BlockedRequests',
    dimensionsMap: {
      WebACL: webAcl.attrArn,
      Rule: 'ALL',
      Region: 'us-east-1',
    },
    statistic: 'Sum',
    period: Duration.minutes(5),
  }),
  threshold: 1000,
  evaluationPeriods: 2,
  comparisonOperator: cloudwatch.ComparisonOperator.GREATER_THAN_THRESHOLD,
});
```

### With AWS Firewall Manager

Use for centralized WAF management across multiple accounts and regions.

### With AWS Shield Advanced

Combine with Shield Advanced for DDoS protection and cost protection guarantees.

### With AWS Security Hub

Integrate findings for centralized security monitoring.

## Migration from WAF Classic

Key differences:
- v2 uses JSON-based rule syntax
- Capacity units (WCU) replaced rule limits
- New features: CAPTCHA, Challenge, label matching
- Improved rule testing with Count mode

Migration steps:
1. Review existing Classic rules
2. Create v2 Web ACL with equivalent rules
3. Test in Count mode
4. Switch traffic to v2 Web ACL
5. Monitor and tune
6. Decommission Classic Web ACL

## Resources

- [AWS WAF Developer Guide](https://docs.aws.amazon.com/waf/latest/developerguide/)
- [AWS Managed Rules List](https://docs.aws.amazon.com/waf/latest/developerguide/aws-managed-rule-groups-list.html)
- [WAF Pricing](https://aws.amazon.com/waf/pricing/)
- [WAF Security Automations](https://aws.amazon.com/solutions/implementations/aws-waf-security-automations/)

## Example: Complete Production Web ACL

```typescript
import * as wafv2 from 'aws-cdk-lib/aws-wafv2';
import { Construct } from 'constructs';

export class ProductionWebAcl extends Construct {
  public readonly webAcl: wafv2.CfnWebACL;

  constructor(scope: Construct, id: string) {
    super(scope, id);

    // IP whitelist for admin endpoints
    const adminIpSet = new wafv2.CfnIPSet(this, 'AdminIPs', {
      scope: 'CLOUDFRONT',
      ipAddressVersion: 'IPV4',
      addresses: ['203.0.113.0/24'],
    });

    this.webAcl = new wafv2.CfnWebACL(this, 'WebAcl', {
      scope: 'CLOUDFRONT',
      defaultAction: { allow: {} },
      rules: [
        // Priority 0: Allow admin IPs
        {
          name: 'AllowAdminIPs',
          priority: 0,
          statement: {
            andStatement: {
              statements: [
                {
                  ipSetReferenceStatement: {
                    arn: adminIpSet.attrArn,
                  },
                },
                {
                  byteMatchStatement: {
                    searchString: '/admin',
                    fieldToMatch: { uriPath: {} },
                    textTransformations: [{ priority: 0, type: 'LOWERCASE' }],
                    positionalConstraint: 'STARTS_WITH',
                  },
                },
              ],
            },
          },
          action: { allow: {} },
          visibilityConfig: {
            sampledRequestsEnabled: true,
            cloudWatchMetricsEnabled: true,
            metricName: 'AllowAdminIPs',
          },
        },
        // Priority 100: Global rate limit
        {
          name: 'GlobalRateLimit',
          priority: 100,
          statement: {
            rateBasedStatement: {
              limit: 2000,
              aggregateKeyType: 'IP',
            },
          },
          action: { block: {} },
          visibilityConfig: {
            sampledRequestsEnabled: true,
            cloudWatchMetricsEnabled: true,
            metricName: 'GlobalRateLimit',
          },
        },
        // Priority 200: Geo blocking
        {
          name: 'GeoBlock',
          priority: 200,
          statement: {
            geoMatchStatement: {
              countryCodes: ['CN', 'RU'],
            },
          },
          action: { block: {} },
          visibilityConfig: {
            sampledRequestsEnabled: true,
            cloudWatchMetricsEnabled: true,
            metricName: 'GeoBlock',
          },
        },
        // Priority 400: AWS Managed Rules - Core Rule Set
        {
          name: 'AWSManagedRulesCommonRuleSet',
          priority: 400,
          statement: {
            managedRuleGroupStatement: {
              vendorName: 'AWS',
              name: 'AWSManagedRulesCommonRuleSet',
            },
          },
          overrideAction: { none: {} },
          visibilityConfig: {
            sampledRequestsEnabled: true,
            cloudWatchMetricsEnabled: true,
            metricName: 'AWSManagedRulesCommonRuleSet',
          },
        },
        // Priority 401: Known Bad Inputs
        {
          name: 'AWSManagedRulesKnownBadInputsRuleSet',
          priority: 401,
          statement: {
            managedRuleGroupStatement: {
              vendorName: 'AWS',
              name: 'AWSManagedRulesKnownBadInputsRuleSet',
            },
          },
          overrideAction: { none: {} },
          visibilityConfig: {
            sampledRequestsEnabled: true,
            cloudWatchMetricsEnabled: true,
            metricName: 'AWSManagedRulesKnownBadInputsRuleSet',
          },
        },
        // Priority 402: Bot Control
        {
          name: 'AWSManagedRulesBotControlRuleSet',
          priority: 402,
          statement: {
            managedRuleGroupStatement: {
              vendorName: 'AWS',
              name: 'AWSManagedRulesBotControlRuleSet',
            },
          },
          overrideAction: { none: {} },
          visibilityConfig: {
            sampledRequestsEnabled: true,
            cloudWatchMetricsEnabled: true,
            metricName: 'AWSManagedRulesBotControlRuleSet',
          },
        },
      ],
      visibilityConfig: {
        sampledRequestsEnabled: true,
        cloudWatchMetricsEnabled: true,
        metricName: 'ProductionWebAcl',
      },
    });
  }
}
```

## Instructions for Using This Skill

When AWS WAF tasks arise:

1. **Assess Requirements**: Determine the use case (CloudFront, ALB, API Gateway)
2. **Choose Scope**: CLOUDFRONT or REGIONAL based on resource
3. **Start Simple**: Begin with AWS Managed Rules
4. **Layer Protection**: Add rate limiting, geo-blocking, IP controls as needed
5. **Test in Count Mode**: Always test before blocking
6. **Enable Logging**: Configure comprehensive logging from the start
7. **Monitor & Tune**: Review metrics and logs regularly
8. **Document**: Keep clear documentation of all custom rules and exceptions

Remember: WAF is about balancing security and usability. Start restrictive, then tune for false positives.
