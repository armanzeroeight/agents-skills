# Parameters

## Contents
- Parameter types and validation
- Parameter design patterns
- Default values and constraints
- AWS-specific parameter types
- SSM parameter integration
- Secrets management
- Best practices

## Parameter Types

### String Parameters

**Basic string:**
```yaml
Parameters:
  ApplicationName:
    Type: String
    Description: Name of the application
    Default: MyApp
```

**With length constraints:**
```yaml
Parameters:
  DatabaseName:
    Type: String
    Description: Database name
    MinLength: 1
    MaxLength: 64
    Default: mydb
```

**With pattern validation:**
```yaml
Parameters:
  DatabaseName:
    Type: String
    Description: Database name (alphanumeric only)
    AllowedPattern: '[a-zA-Z][a-zA-Z0-9]*'
    ConstraintDescription: Must begin with letter, contain only alphanumeric characters
```

### Number Parameters

**Integer:**
```yaml
Parameters:
  InstanceCount:
    Type: Number
    Description: Number of instances
    Default: 2
    MinValue: 1
    MaxValue: 10
```

**With specific values:**
```yaml
Parameters:
  Port:
    Type: Number
    Description: Application port
    Default: 8080
    AllowedValues: [80, 443, 8080, 8443]
```

### List Parameters

**Comma-delimited list:**
```yaml
Parameters:
  AvailabilityZones:
    Type: CommaDelimitedList
    Description: Availability zones (comma-separated)
    Default: 'us-east-1a,us-east-1b'
```

**Using list in resources:**
```yaml
Resources:
  AutoScalingGroup:
    Type: AWS::AutoScaling::AutoScalingGroup
    Properties:
      AvailabilityZones: !Ref AvailabilityZones
```

### AWS-Specific Types

**VPC ID:**
```yaml
Parameters:
  VpcId:
    Type: AWS::EC2::VPC::Id
    Description: VPC for resources
```

**Subnet IDs:**
```yaml
Parameters:
  SubnetIds:
    Type: List<AWS::EC2::Subnet::Id>
    Description: Subnets for resources
```

**Security Group IDs:**
```yaml
Parameters:
  SecurityGroupIds:
    Type: List<AWS::EC2::SecurityGroup::Id>
    Description: Security groups
```

**Key pair name:**
```yaml
Parameters:
  KeyName:
    Type: AWS::EC2::KeyPair::KeyName
    Description: EC2 key pair
```

**AMI ID:**
```yaml
Parameters:
  ImageId:
    Type: AWS::EC2::Image::Id
    Description: AMI ID
```

**Availability Zone:**
```yaml
Parameters:
  AvailabilityZone:
    Type: AWS::EC2::AvailabilityZone::Name
    Description: Availability zone
```

## Parameter Validation

### Allowed Values

```yaml
Parameters:
  Environment:
    Type: String
    Description: Environment name
    Default: dev
    AllowedValues:
      - dev
      - staging
      - prod
    ConstraintDescription: Must be dev, staging, or prod
```

### Pattern Matching

**Email address:**
```yaml
Parameters:
  AdminEmail:
    Type: String
    Description: Administrator email
    AllowedPattern: '^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    ConstraintDescription: Must be a valid email address
```

**CIDR block:**
```yaml
Parameters:
  VpcCidr:
    Type: String
    Description: VPC CIDR block
    Default: 10.0.0.0/16
    AllowedPattern: '^(([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])\.){3}([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])(\/([0-9]|[1-2][0-9]|3[0-2]))$'
    ConstraintDescription: Must be a valid CIDR block
```

**S3 bucket name:**
```yaml
Parameters:
  BucketName:
    Type: String
    Description: S3 bucket name
    AllowedPattern: '^[a-z0-9][a-z0-9-]*[a-z0-9]$'
    MinLength: 3
    MaxLength: 63
    ConstraintDescription: Must be valid S3 bucket name (lowercase, numbers, hyphens)
```

### Range Validation

```yaml
Parameters:
  DatabaseStorage:
    Type: Number
    Description: Database storage in GB
    Default: 20
    MinValue: 20
    MaxValue: 1000
    ConstraintDescription: Must be between 20 and 1000 GB
```

## Default Values

### Static Defaults

```yaml
Parameters:
  InstanceType:
    Type: String
    Description: EC2 instance type
    Default: t3.micro
    AllowedValues:
      - t3.micro
      - t3.small
      - t3.medium
```

### Environment-Based Defaults

Use mappings for environment-specific defaults:

```yaml
Parameters:
  Environment:
    Type: String
    Default: dev
    AllowedValues: [dev, staging, prod]

Mappings:
  EnvironmentDefaults:
    dev:
      InstanceType: t3.micro
      MinSize: 1
      MaxSize: 2
    staging:
      InstanceType: t3.small
      MinSize: 2
      MaxSize: 4
    prod:
      InstanceType: t3.medium
      MinSize: 3
      MaxSize: 10

Resources:
  LaunchTemplate:
    Type: AWS::EC2::LaunchTemplate
    Properties:
      LaunchTemplateData:
        InstanceType: !FindInMap [EnvironmentDefaults, !Ref Environment, InstanceType]
```

## SSM Parameter Integration

### Reading SSM Parameters

**String parameter:**
```yaml
Parameters:
  LatestAmiId:
    Type: AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>
    Default: /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2
    Description: Latest Amazon Linux 2 AMI

Resources:
  Instance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !Ref LatestAmiId
```

**Secure string parameter:**
```yaml
Parameters:
  DatabasePassword:
    Type: AWS::SSM::Parameter::Value<String>
    Default: /myapp/database/password
    NoEcho: true
    Description: Database password from SSM

Resources:
  Database:
    Type: AWS::RDS::DBInstance
    Properties:
      MasterUserPassword: !Ref DatabasePassword
```

**List parameter:**
```yaml
Parameters:
  SubnetIds:
    Type: AWS::SSM::Parameter::Value<List<AWS::EC2::Subnet::Id>>
    Default: /myapp/network/subnet-ids
    Description: Subnet IDs from SSM
```

### Dynamic SSM Resolution

```yaml
Resources:
  Instance:
    Type: AWS::EC2::Instance
    Properties:
      # Resolve SSM parameter at runtime
      ImageId: !Sub '{{resolve:ssm:/aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2}}'
```

## Secrets Management

### Secrets Manager Integration

**Reading secrets:**
```yaml
Resources:
  Database:
    Type: AWS::RDS::DBInstance
    Properties:
      # Resolve secret at runtime
      MasterUserPassword: !Sub '{{resolve:secretsmanager:${DatabaseSecret}:SecretString:password}}'
  
  DatabaseSecret:
    Type: AWS::SecretsManager::Secret
    Properties:
      Description: Database credentials
      GenerateSecretString:
        SecretStringTemplate: '{"username": "admin"}'
        GenerateStringKey: password
        PasswordLength: 32
        ExcludeCharacters: '"@/\'
```

**JSON secret fields:**
```yaml
Resources:
  Application:
    Type: AWS::ECS::TaskDefinition
    Properties:
      ContainerDefinitions:
        - Name: app
          Environment:
            - Name: DB_HOST
              Value: !Sub '{{resolve:secretsmanager:${AppSecret}:SecretString:host}}'
            - Name: DB_USER
              Value: !Sub '{{resolve:secretsmanager:${AppSecret}:SecretString:username}}'
            - Name: DB_PASS
              Value: !Sub '{{resolve:secretsmanager:${AppSecret}:SecretString:password}}'
```

### NoEcho for Sensitive Parameters

```yaml
Parameters:
  DatabasePassword:
    Type: String
    Description: Database password
    NoEcho: true
    MinLength: 8
    MaxLength: 41
    AllowedPattern: '[a-zA-Z0-9]*'
    ConstraintDescription: Must contain only alphanumeric characters
```

## Parameter Design Patterns

### Pattern 1: Required vs Optional

**Required parameter (no default):**
```yaml
Parameters:
  VpcId:
    Type: AWS::EC2::VPC::Id
    Description: VPC ID (required)
```

**Optional parameter (with default):**
```yaml
Parameters:
  EnableMonitoring:
    Type: String
    Description: Enable detailed monitoring
    Default: 'false'
    AllowedValues: ['true', 'false']
```

### Pattern 2: Conditional Parameters

```yaml
Parameters:
  CreateDatabase:
    Type: String
    Default: 'true'
    AllowedValues: ['true', 'false']
  
  DatabaseInstanceClass:
    Type: String
    Default: db.t3.micro
    Description: Only used if CreateDatabase is true

Conditions:
  ShouldCreateDatabase: !Equals [!Ref CreateDatabase, 'true']

Resources:
  Database:
    Type: AWS::RDS::DBInstance
    Condition: ShouldCreateDatabase
    Properties:
      DBInstanceClass: !Ref DatabaseInstanceClass
```

### Pattern 3: Hierarchical Parameters

```yaml
Parameters:
  # Top-level parameter
  Environment:
    Type: String
    AllowedValues: [dev, staging, prod]
  
  # Derived from environment
  EnableBackups:
    Type: String
    Default: 'false'
    AllowedValues: ['true', 'false']
    Description: Override default backup setting

Conditions:
  IsProduction: !Equals [!Ref Environment, 'prod']
  # Enable backups if production OR explicitly enabled
  BackupsEnabled: !Or
    - !Condition IsProduction
    - !Equals [!Ref EnableBackups, 'true']
```

### Pattern 4: Parameter Groups

Organize related parameters:

```yaml
Metadata:
  AWS::CloudFormation::Interface:
    ParameterGroups:
      - Label:
          default: Network Configuration
        Parameters:
          - VpcId
          - SubnetIds
          - SecurityGroupIds
      
      - Label:
          default: Instance Configuration
        Parameters:
          - InstanceType
          - KeyName
          - ImageId
      
      - Label:
          default: Application Configuration
        Parameters:
          - ApplicationName
          - Environment
          - EnableMonitoring
    
    ParameterLabels:
      VpcId:
        default: VPC ID
      SubnetIds:
        default: Subnet IDs
      InstanceType:
        default: Instance Type
```

## Best Practices

### Naming Conventions

**Good names:**
- `VpcId` - Clear and concise
- `DatabaseInstanceClass` - Descriptive
- `EnableDetailedMonitoring` - Boolean intent clear

**Avoid:**
- `Param1` - Not descriptive
- `vpc` - Not capitalized
- `TheVPCIdForTheApplication` - Too verbose

### Description Guidelines

**Good descriptions:**
```yaml
Parameters:
  InstanceType:
    Type: String
    Description: EC2 instance type for application servers
    Default: t3.micro
```

**Include context:**
```yaml
Parameters:
  MaxSize:
    Type: Number
    Description: Maximum number of instances in Auto Scaling group (affects cost)
    Default: 10
```

### Validation Best Practices

**Always validate:**
```yaml
Parameters:
  DatabaseName:
    Type: String
    Description: Database name
    MinLength: 1
    MaxLength: 64
    AllowedPattern: '[a-zA-Z][a-zA-Z0-9]*'
    ConstraintDescription: Must begin with letter, contain only alphanumeric
```

**Provide helpful constraint messages:**
```yaml
Parameters:
  CidrBlock:
    Type: String
    AllowedPattern: '^(([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])\.){3}([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])(\/([0-9]|[1-2][0-9]|3[0-2]))$'
    ConstraintDescription: Must be a valid IPv4 CIDR block (e.g., 10.0.0.0/16)
```

### Security Best Practices

**Use NoEcho for sensitive data:**
```yaml
Parameters:
  ApiKey:
    Type: String
    NoEcho: true
    Description: API key (will not be displayed)
```

**Prefer SSM/Secrets Manager:**
```yaml
# Better than passing secrets as parameters
Parameters:
  DatabasePasswordArn:
    Type: String
    Description: ARN of secret containing database password

Resources:
  Database:
    Type: AWS::RDS::DBInstance
    Properties:
      MasterUserPassword: !Sub '{{resolve:secretsmanager:${DatabasePasswordArn}:SecretString:password}}'
```

**Don't use defaults for secrets:**
```yaml
# Bad - exposes secret in template
Parameters:
  ApiKey:
    Type: String
    Default: 'my-secret-key'  # Don't do this!

# Good - require user to provide
Parameters:
  ApiKey:
    Type: String
    NoEcho: true
    Description: API key (required)
```

## Common Patterns

### Pattern: Environment Configuration

```yaml
Parameters:
  Environment:
    Type: String
    Default: dev
    AllowedValues: [dev, staging, prod]
    Description: Environment name

Mappings:
  EnvironmentConfig:
    dev:
      InstanceType: t3.micro
      MinSize: 1
      MaxSize: 2
      EnableBackups: false
      EnableMonitoring: false
    staging:
      InstanceType: t3.small
      MinSize: 2
      MaxSize: 4
      EnableBackups: true
      EnableMonitoring: false
    prod:
      InstanceType: t3.medium
      MinSize: 3
      MaxSize: 10
      EnableBackups: true
      EnableMonitoring: true

Resources:
  AutoScalingGroup:
    Type: AWS::AutoScaling::AutoScalingGroup
    Properties:
      MinSize: !FindInMap [EnvironmentConfig, !Ref Environment, MinSize]
      MaxSize: !FindInMap [EnvironmentConfig, !Ref Environment, MaxSize]
```

### Pattern: Feature Flags

```yaml
Parameters:
  EnableCaching:
    Type: String
    Default: 'false'
    AllowedValues: ['true', 'false']
  
  EnableLogging:
    Type: String
    Default: 'true'
    AllowedValues: ['true', 'false']
  
  EnableMonitoring:
    Type: String
    Default: 'false'
    AllowedValues: ['true', 'false']

Conditions:
  CachingEnabled: !Equals [!Ref EnableCaching, 'true']
  LoggingEnabled: !Equals [!Ref EnableLogging, 'true']
  MonitoringEnabled: !Equals [!Ref EnableMonitoring, 'true']

Resources:
  CacheCluster:
    Type: AWS::ElastiCache::CacheCluster
    Condition: CachingEnabled
    Properties:
      # ...
```

### Pattern: Multi-Region Support

```yaml
Parameters:
  PrimaryRegion:
    Type: String
    Default: us-east-1
    AllowedValues:
      - us-east-1
      - us-west-2
      - eu-west-1
  
  SecondaryRegion:
    Type: String
    Default: us-west-2
    AllowedValues:
      - us-east-1
      - us-west-2
      - eu-west-1

Mappings:
  RegionConfig:
    us-east-1:
      AvailabilityZones: us-east-1a,us-east-1b,us-east-1c
    us-west-2:
      AvailabilityZones: us-west-2a,us-west-2b,us-west-2c
    eu-west-1:
      AvailabilityZones: eu-west-1a,eu-west-1b,eu-west-1c
```

## Troubleshooting

### Common Issues

**Issue: Parameter validation fails**
```
Error: Parameter validation failed: Value does not match pattern
```

**Solution:**
- Check AllowedPattern regex
- Verify input matches pattern
- Review ConstraintDescription for guidance

**Issue: SSM parameter not found**
```
Error: Parameter /path/to/parameter does not exist
```

**Solution:**
- Verify parameter exists in SSM
- Check parameter path spelling
- Ensure correct region
- Verify IAM permissions

**Issue: Circular dependency with parameters**
```
Error: Circular dependency between resources
```

**Solution:**
- Review parameter references
- Check if parameters reference each other
- Simplify parameter dependencies

### Debugging Parameters

**View parameter values:**
```bash
aws cloudformation describe-stacks \
  --stack-name my-stack \
  --query 'Stacks[0].Parameters'
```

**Test parameter validation:**
```bash
aws cloudformation validate-template \
  --template-body file://template.yaml \
  --parameters ParameterKey=Param1,ParameterValue=Value1
```

## Limits

**CloudFormation parameter limits:**
- Maximum 200 parameters per template
- Parameter name: 255 characters max
- Parameter value: 4096 characters max
- Description: 4000 characters max
- AllowedPattern: 4096 characters max
