---
name: template-validator
description: Validates CloudFormation templates for syntax, security, and best practices. Use when validating CloudFormation templates, checking for security issues, or ensuring compliance with best practices.
---

# Template Validator

## Quick Start

Validate CloudFormation templates for syntax errors, security issues, and adherence to best practices before deployment.

## Instructions

### Step 1: Validate template syntax

```bash
# Basic validation
aws cloudformation validate-template \
  --template-body file://template.yaml

# Validation with parameters
aws cloudformation validate-template \
  --template-body file://template.yaml \
  --parameters ParameterKey=Param1,ParameterValue=Value1
```

**Check for:**
- Valid YAML/JSON syntax
- Required template sections
- Valid resource types
- Correct intrinsic function usage
- Parameter references

### Step 2: Use cfn-lint for comprehensive checks

```bash
# Install cfn-lint
pip install cfn-lint

# Validate template
cfn-lint template.yaml

# Validate with specific rules
cfn-lint template.yaml --ignore-checks W

# Output as JSON
cfn-lint template.yaml --format json
```

**cfn-lint checks:**
- Template structure
- Resource properties
- Best practices
- Security issues
- Regional availability

### Step 3: Security validation

**Check IAM policies:**
```yaml
# Review for overly permissive policies
Resources:
  Role:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Statement:
          - Effect: Allow
            Principal:
              Service: ec2.amazonaws.com
            Action: sts:AssumeRole
      Policies:
        - PolicyName: AppPolicy
          PolicyDocument:
            Statement:
              # Avoid wildcards
              - Effect: Allow
                Action: s3:*  # Too permissive!
                Resource: '*'  # Too broad!
```

**Better approach:**
```yaml
Policies:
  - PolicyName: AppPolicy
    PolicyDocument:
      Statement:
        - Effect: Allow
          Action:
            - s3:GetObject
            - s3:PutObject
          Resource: !Sub '${MyBucket.Arn}/*'
```

**Check security groups:**
```yaml
# Avoid open access
Resources:
  SecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      SecurityGroupIngress:
        # Don't allow 0.0.0.0/0 for SSH
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0  # Security risk!
```

**Better approach:**
```yaml
SecurityGroupIngress:
  - IpProtocol: tcp
    FromPort: 22
    ToPort: 22
    CidrIp: 10.0.0.0/8  # Restrict to internal network
```

### Step 4: Check resource dependencies

**Verify DependsOn usage:**
```yaml
Resources:
  # Explicit dependency needed
  Instance:
    Type: AWS::EC2::Instance
    DependsOn: InternetGatewayAttachment
    Properties:
      # ...
  
  # Implicit dependency via Ref
  SecurityGroupIngress:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !Ref SecurityGroup  # Implicit dependency
```

**Check for circular dependencies:**
- Review all DependsOn relationships
- Check Ref and GetAtt usage
- Verify no circular references

### Step 5: Validate best practices

**Use specific resource names:**
```yaml
# Good
Resources:
  WebServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup

# Avoid generic names
Resources:
  SecurityGroup1:
    Type: AWS::EC2::SecurityGroup
```

**Add descriptions:**
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Web application infrastructure with ALB and Auto Scaling

Parameters:
  InstanceType:
    Type: String
    Description: EC2 instance type for web servers
```

**Use tags:**
```yaml
Resources:
  Instance:
    Type: AWS::EC2::Instance
    Properties:
      Tags:
        - Key: Name
          Value: !Sub '${AWS::StackName}-WebServer'
        - Key: Environment
          Value: !Ref Environment
        - Key: ManagedBy
          Value: CloudFormation
```

## Common Validation Checks

### Syntax Validation

**Valid YAML structure:**
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Template description

Parameters:
  # Parameters section

Resources:
  # Resources section (required)

Outputs:
  # Outputs section
```

**Intrinsic functions:**
```yaml
# Correct
Value: !Ref MyResource
Value: !GetAtt MyResource.Attribute
Value: !Sub '${MyResource}'

# Incorrect
Value: Ref: MyResource  # Wrong syntax
Value: !GetAtt MyResource  # Missing attribute
```

### Security Validation

**IAM policies:**
- No wildcards in actions unless necessary
- Specific resources instead of '*'
- Least privilege principle
- No hardcoded credentials

**Security groups:**
- No 0.0.0.0/0 for sensitive ports (22, 3389, 3306, 5432)
- Specific port ranges
- Documented ingress rules

**Encryption:**
- Enable encryption for S3 buckets
- Enable encryption for EBS volumes
- Enable encryption for RDS instances
- Use KMS keys for sensitive data

### Resource Validation

**Required properties:**
```yaml
Resources:
  Bucket:
    Type: AWS::S3::Bucket
    Properties:
      # BucketName is optional but recommended
      BucketName: !Sub '${AWS::StackName}-bucket'
```

**Valid property values:**
```yaml
Resources:
  Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t3.micro  # Must be valid instance type
      ImageId: ami-12345678  # Must be valid AMI ID
```

## Validation Tools

### AWS CLI

```bash
# Validate template
aws cloudformation validate-template \
  --template-body file://template.yaml

# Create change set (validates before applying)
aws cloudformation create-change-set \
  --stack-name my-stack \
  --change-set-name my-changes \
  --template-body file://template.yaml

# Describe change set
aws cloudformation describe-change-set \
  --stack-name my-stack \
  --change-set-name my-changes
```

### cfn-lint

```bash
# Basic validation
cfn-lint template.yaml

# Ignore warnings
cfn-lint template.yaml --ignore-checks W

# Specific regions
cfn-lint template.yaml --regions us-east-1 us-west-2

# Custom rules
cfn-lint template.yaml --append-rules custom-rules/
```

### cfn-nag

```bash
# Install cfn-nag
gem install cfn-nag

# Scan template
cfn_nag_scan --input-path template.yaml

# Scan with rules
cfn_nag_scan --input-path template.yaml --deny-list-path rules.txt
```

### TaskCat

```bash
# Install taskcat
pip install taskcat

# Test template
taskcat test run

# Configuration in .taskcat.yml
project:
  name: my-project
  regions:
    - us-east-1
    - us-west-2
tests:
  default:
    template: template.yaml
    parameters:
      InstanceType: t3.micro
```

## Validation Checklist

**Template structure:**
- [ ] Valid YAML/JSON syntax
- [ ] AWSTemplateFormatVersion present
- [ ] Description provided
- [ ] Resources section present

**Parameters:**
- [ ] Descriptive names
- [ ] Descriptions provided
- [ ] Validation constraints (AllowedValues, AllowedPattern)
- [ ] Appropriate defaults
- [ ] NoEcho for sensitive values

**Resources:**
- [ ] Descriptive logical IDs
- [ ] Required properties present
- [ ] Valid property values
- [ ] Appropriate DependsOn usage
- [ ] Tags applied

**Security:**
- [ ] IAM policies follow least privilege
- [ ] No hardcoded credentials
- [ ] Security groups restrict access
- [ ] Encryption enabled where appropriate
- [ ] No overly permissive policies

**Outputs:**
- [ ] Descriptive names
- [ ] Descriptions provided
- [ ] Appropriate exports
- [ ] Conditional outputs where needed

**Best practices:**
- [ ] Consistent naming convention
- [ ] Appropriate use of parameters
- [ ] Cross-stack references via exports
- [ ] Proper error handling
- [ ] Documentation in descriptions

## Advanced

For detailed information, see:
- [Security Best Practices](reference/security-best-practices.md) - Comprehensive security validation guide
- [Validation Rules](reference/validation-rules.md) - Complete list of validation rules and checks
