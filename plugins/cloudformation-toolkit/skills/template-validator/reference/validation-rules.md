# Validation Rules

## Contents
- Template structure rules
- Resource property rules
- Security rules
- Best practice rules
- Performance rules

## Template Structure Rules

### Required Sections

**Rule: Template must have Resources section**
```yaml
# Valid
AWSTemplateFormatVersion: '2010-09-09'
Resources:
  MyBucket:
    Type: AWS::S3::Bucket

# Invalid - missing Resources
AWSTemplateFormatVersion: '2010-09-09'
Parameters:
  Param1:
    Type: String
```

### Template Format Version

**Rule: Use correct format version**
```yaml
# Valid
AWSTemplateFormatVersion: '2010-09-09'

# Invalid
AWSTemplateFormatVersion: '2010-09-10'  # Wrong version
```

### Description

**Rule: Provide template description**
```yaml
# Good
AWSTemplateFormatVersion: '2010-09-09'
Description: Web application infrastructure with ALB and Auto Scaling

# Acceptable but not recommended
AWSTemplateFormatVersion: '2010-09-09'
# No description
```

## Resource Property Rules

### Required Properties

**Rule: Include all required properties**
```yaml
# Valid
Resources:
  Instance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-12345678  # Required
      InstanceType: t3.micro  # Required

# Invalid - missing required properties
Resources:
  Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t3.micro  # Missing ImageId
```

### Property Types

**Rule: Use correct property types**
```yaml
# Valid
Resources:
  Bucket:
    Type: AWS::S3::Bucket
    Properties:
      VersioningConfiguration:
        Status: Enabled  # String

# Invalid
Resources:
  Bucket:
    Type: AWS::S3::Bucket
    Properties:
      VersioningConfiguration:
        Status: true  # Should be string, not boolean
```

### Property Values

**Rule: Use valid property values**
```yaml
# Valid
Resources:
  Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t3.micro  # Valid instance type

# Invalid
Resources:
  Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: invalid-type  # Not a valid instance type
```

## Security Rules

### IAM Policies

**Rule: No wildcard actions with wildcard resources**
```yaml
# Violation
Policies:
  - PolicyDocument:
      Statement:
        - Effect: Allow
          Action: '*'
          Resource: '*'

# Compliant
Policies:
  - PolicyDocument:
      Statement:
        - Effect: Allow
          Action:
            - s3:GetObject
            - s3:PutObject
          Resource: !Sub '${Bucket.Arn}/*'
```

### Security Groups

**Rule: No unrestricted SSH/RDP access**
```yaml
# Violation
SecurityGroupIngress:
  - IpProtocol: tcp
    FromPort: 22
    ToPort: 22
    CidrIp: 0.0.0.0/0

# Compliant
SecurityGroupIngress:
  - IpProtocol: tcp
    FromPort: 22
    ToPort: 22
    CidrIp: 10.0.0.0/8
```

**Rule: No unrestricted database access**
```yaml
# Violation
SecurityGroupIngress:
  - IpProtocol: tcp
    FromPort: 3306
    ToPort: 3306
    CidrIp: 0.0.0.0/0

# Compliant
SecurityGroupIngress:
  - IpProtocol: tcp
    FromPort: 3306
    ToPort: 3306
    SourceSecurityGroupId: !Ref AppSecurityGroup
```

### Encryption

**Rule: Enable S3 bucket encryption**
```yaml
# Violation
Resources:
  Bucket:
    Type: AWS::S3::Bucket

# Compliant
Resources:
  Bucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketEncryption:
        ServerSideEncryptionConfiguration:
          - ServerSideEncryptionByDefault:
              SSEAlgorithm: AES256
```

**Rule: Enable RDS encryption**
```yaml
# Violation
Resources:
  Database:
    Type: AWS::RDS::DBInstance
    Properties:
      DBInstanceClass: db.t3.micro

# Compliant
Resources:
  Database:
    Type: AWS::RDS::DBInstance
    Properties:
      DBInstanceClass: db.t3.micro
      StorageEncrypted: true
```

### Secrets

**Rule: No hardcoded credentials**
```yaml
# Violation
Resources:
  Database:
    Type: AWS::RDS::DBInstance
    Properties:
      MasterUserPassword: 'password123'

# Compliant
Resources:
  Database:
    Type: AWS::RDS::DBInstance
    Properties:
      MasterUserPassword: !Sub '{{resolve:secretsmanager:${Secret}:SecretString:password}}'
```

## Best Practice Rules

### Naming

**Rule: Use descriptive resource names**
```yaml
# Good
Resources:
  WebServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
  
  DatabaseSubnetGroup:
    Type: AWS::RDS::DBSubnetGroup

# Avoid
Resources:
  SG1:
    Type: AWS::EC2::SecurityGroup
  
  SubnetGroup:
    Type: AWS::RDS::DBSubnetGroup
```

### Tagging

**Rule: Apply tags to resources**
```yaml
# Good
Resources:
  Instance:
    Type: AWS::EC2::Instance
    Properties:
      Tags:
        - Key: Name
          Value: !Sub '${AWS::StackName}-WebServer'
        - Key: Environment
          Value: !Ref Environment

# Missing tags
Resources:
  Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t3.micro
```

### Parameters

**Rule: Validate parameter inputs**
```yaml
# Good
Parameters:
  InstanceType:
    Type: String
    AllowedValues: [t3.micro, t3.small, t3.medium]
    Default: t3.micro

# Missing validation
Parameters:
  InstanceType:
    Type: String
```

### Outputs

**Rule: Provide output descriptions**
```yaml
# Good
Outputs:
  VpcId:
    Description: VPC ID for application resources
    Value: !Ref VPC

# Missing description
Outputs:
  VpcId:
    Value: !Ref VPC
```

## Performance Rules

### Resource Limits

**Rule: Stay within CloudFormation limits**
- Maximum 500 resources per template
- Maximum 200 parameters
- Maximum 200 outputs
- Maximum 200 mappings

### DependsOn Usage

**Rule: Use DependsOn only when necessary**
```yaml
# Necessary - explicit dependency
Resources:
  Instance:
    Type: AWS::EC2::Instance
    DependsOn: InternetGatewayAttachment

# Unnecessary - implicit via Ref
Resources:
  SecurityGroupIngress:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !Ref SecurityGroup  # Implicit dependency
```

### Circular Dependencies

**Rule: Avoid circular dependencies**
```yaml
# Violation - circular dependency
Resources:
  Resource1:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !GetAtt Resource2.Arn
  
  Resource2:
    Type: AWS::S3::Bucket
    DependsOn: Resource1

# Compliant - no circular dependency
Resources:
  Resource1:
    Type: AWS::S3::Bucket
  
  Resource2:
    Type: AWS::S3::Bucket
    DependsOn: Resource1
```

## Validation Tools and Rules

### cfn-lint Rules

**Error rules (E):**
- E1001: Basic CloudFormation template error
- E1010: Invalid template version
- E2001: Missing required property
- E3001: Invalid property value

**Warning rules (W):**
- W2001: Parameter not used
- W3001: Hardcoded resource name
- W3005: DependsOn not needed

**Informational rules (I):**
- I3011: Suggest using specific resource types

### cfn-nag Rules

**Security rules:**
- F1: IAM policy with wildcard action and resource
- F2: Security group with unrestricted ingress
- F3: RDS instance without encryption
- F4: S3 bucket without encryption

**Warning rules:**
- W1: Security group with unrestricted egress
- W2: IAM policy with NotAction
- W3: Missing CloudTrail log validation

## Custom Validation Rules

### Organization-Specific Rules

**Example: Require specific tags**
```python
# Custom cfn-lint rule
from cfnlint.rules import CloudFormationLintRule

class RequireEnvironmentTag(CloudFormationLintRule):
    id = 'E9001'
    shortdesc = 'Resources must have Environment tag'
    
    def match(self, cfn):
        matches = []
        resources = cfn.get_resources()
        
        for resource_name, resource in resources.items():
            tags = resource.get('Properties', {}).get('Tags', [])
            if not any(tag.get('Key') == 'Environment' for tag in tags):
                matches.append(...)
        
        return matches
```

### Compliance Rules

**Example: Enforce encryption**
```yaml
# Policy as code
Rules:
  S3BucketEncryption:
    Assertions:
      - Assert: !Equals [!GetAtt Bucket.BucketEncryption, 'Enabled']
        AssertDescription: S3 buckets must have encryption enabled
```

## Validation Workflow

### Pre-Deployment Validation

1. **Syntax validation**
   ```bash
   aws cloudformation validate-template --template-body file://template.yaml
   ```

2. **Linting**
   ```bash
   cfn-lint template.yaml
   ```

3. **Security scanning**
   ```bash
   cfn_nag_scan --input-path template.yaml
   ```

4. **Change set review**
   ```bash
   aws cloudformation create-change-set --stack-name my-stack --change-set-name my-changes --template-body file://template.yaml
   aws cloudformation describe-change-set --stack-name my-stack --change-set-name my-changes
   ```

### Continuous Validation

**CI/CD pipeline:**
```yaml
# .github/workflows/validate.yml
name: Validate CloudFormation
on: [push, pull_request]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Install tools
        run: |
          pip install cfn-lint
          gem install cfn-nag
      
      - name: Validate syntax
        run: |
          aws cloudformation validate-template --template-body file://template.yaml
      
      - name: Lint template
        run: |
          cfn-lint template.yaml
      
      - name: Security scan
        run: |
          cfn_nag_scan --input-path template.yaml
```

## Rule Exceptions

### Suppressing Rules

**cfn-lint:**
```yaml
# Suppress specific rule
Resources:
  Bucket:
    Type: AWS::S3::Bucket
    Metadata:
      cfn-lint:
        config:
          ignore_checks:
            - W3001
```

**cfn-nag:**
```yaml
# Suppress specific rule
Resources:
  SecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Metadata:
      cfn_nag:
        rules_to_suppress:
          - id: F1000
            reason: Public access required for web tier
```

### When to Suppress

- False positives
- Intentional design decisions
- Temporary exceptions (with tracking)
- Legacy compatibility

**Document suppressions:**
```yaml
Metadata:
  cfn_nag:
    rules_to_suppress:
      - id: W2
        reason: NotAction required for service control policy
      - id: F3
        reason: Encryption not supported in this region (temporary)
```
