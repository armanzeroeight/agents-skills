---
name: stack-designer
description: Designs CloudFormation stack structure, nested stacks, and resource organization. Use when designing CloudFormation infrastructure, organizing resources into stacks, or planning nested stack hierarchies.
---

# Stack Designer

## Quick Start

Design well-organized CloudFormation stacks with proper resource grouping, parameters, outputs, and cross-stack references.

## Instructions

### Step 1: Identify stack boundaries

Determine how to organize resources into stacks:

**By lifecycle:**
- Resources that change together should be in the same stack
- Separate frequently updated resources from stable infrastructure
- Group by deployment frequency

**By ownership:**
- Network stack (VPC, subnets, route tables)
- Security stack (security groups, IAM roles)
- Application stack (EC2, ECS, Lambda)
- Data stack (RDS, DynamoDB, S3)

**By environment:**
- Separate dev, staging, production stacks
- Use parameters for environment-specific values
- Share common resources via cross-stack references

### Step 2: Design stack structure

**Simple stack (single template):**
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Simple web application stack

Parameters:
  EnvironmentName:
    Type: String
    Default: dev
    AllowedValues: [dev, staging, prod]
  
  InstanceType:
    Type: String
    Default: t3.micro
    AllowedValues: [t3.micro, t3.small, t3.medium]

Resources:
  WebServer:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref InstanceType
      ImageId: !Sub '{{resolve:ssm:/aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2}}'
      Tags:
        - Key: Environment
          Value: !Ref EnvironmentName

Outputs:
  InstanceId:
    Description: EC2 instance ID
    Value: !Ref WebServer
    Export:
      Name: !Sub '${AWS::StackName}-InstanceId'
```

**Multi-stack architecture:**
```
Root Stack
├── Network Stack (VPC, subnets)
├── Security Stack (security groups, IAM)
├── Database Stack (RDS)
└── Application Stack (EC2, ALB)
```

### Step 3: Define parameters

**Parameter best practices:**
```yaml
Parameters:
  # Use descriptive names
  DatabaseInstanceClass:
    Type: String
    Default: db.t3.micro
    AllowedValues:
      - db.t3.micro
      - db.t3.small
      - db.t3.medium
    Description: RDS instance class
  
  # Validate input
  DatabaseName:
    Type: String
    MinLength: 1
    MaxLength: 64
    AllowedPattern: '[a-zA-Z][a-zA-Z0-9]*'
    ConstraintDescription: Must begin with letter, contain only alphanumeric
  
  # Use AWS-specific types
  VpcId:
    Type: AWS::EC2::VPC::Id
    Description: VPC for resources
  
  SubnetIds:
    Type: List<AWS::EC2::Subnet::Id>
    Description: Subnets for resources
  
  # Sensitive values from SSM
  DatabasePassword:
    Type: AWS::SSM::Parameter::Value<String>
    Default: /myapp/database/password
    NoEcho: true
```

### Step 4: Configure outputs

**Output best practices:**
```yaml
Outputs:
  # Export for cross-stack references
  VpcId:
    Description: VPC ID
    Value: !Ref VPC
    Export:
      Name: !Sub '${AWS::StackName}-VpcId'
  
  # Multiple values
  PrivateSubnetIds:
    Description: Private subnet IDs
    Value: !Join [',', [!Ref PrivateSubnet1, !Ref PrivateSubnet2]]
    Export:
      Name: !Sub '${AWS::StackName}-PrivateSubnets'
  
  # Resource attributes
  LoadBalancerDNS:
    Description: ALB DNS name
    Value: !GetAtt ApplicationLoadBalancer.DNSName
  
  # Conditional outputs
  DatabaseEndpoint:
    Condition: CreateDatabase
    Description: RDS endpoint
    Value: !GetAtt Database.Endpoint.Address
```

### Step 5: Implement cross-stack references

**Exporting from one stack:**
```yaml
Outputs:
  SecurityGroupId:
    Value: !Ref WebSecurityGroup
    Export:
      Name: !Sub '${AWS::StackName}-SecurityGroupId'
```

**Importing in another stack:**
```yaml
Resources:
  WebServer:
    Type: AWS::EC2::Instance
    Properties:
      SecurityGroupIds:
        - !ImportValue NetworkStack-SecurityGroupId
```

## Nested Stacks

**Parent stack:**
```yaml
Resources:
  NetworkStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/mybucket/network.yaml
      Parameters:
        EnvironmentName: !Ref EnvironmentName
      Tags:
        - Key: Name
          Value: Network
  
  ApplicationStack:
    Type: AWS::CloudFormation::Stack
    DependsOn: NetworkStack
    Properties:
      TemplateURL: https://s3.amazonaws.com/mybucket/application.yaml
      Parameters:
        VpcId: !GetAtt NetworkStack.Outputs.VpcId
        SubnetIds: !GetAtt NetworkStack.Outputs.SubnetIds
```

**Benefits:**
- Reusable templates
- Logical organization
- Independent updates
- Overcome template size limits

## Common Patterns

### Pattern 1: Environment-specific stacks

```yaml
# Use parameters for environment differences
Parameters:
  Environment:
    Type: String
    AllowedValues: [dev, staging, prod]

Mappings:
  EnvironmentConfig:
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
  AutoScalingGroup:
    Type: AWS::AutoScaling::AutoScalingGroup
    Properties:
      MinSize: !FindInMap [EnvironmentConfig, !Ref Environment, MinSize]
      MaxSize: !FindInMap [EnvironmentConfig, !Ref Environment, MaxSize]
      LaunchTemplate:
        LaunchTemplateId: !Ref LaunchTemplate
        Version: !GetAtt LaunchTemplate.LatestVersionNumber
```

### Pattern 2: Conditional resources

```yaml
Parameters:
  CreateDatabase:
    Type: String
    Default: 'true'
    AllowedValues: ['true', 'false']

Conditions:
  ShouldCreateDatabase: !Equals [!Ref CreateDatabase, 'true']
  IsProduction: !Equals [!Ref Environment, 'prod']

Resources:
  Database:
    Type: AWS::RDS::DBInstance
    Condition: ShouldCreateDatabase
    Properties:
      DBInstanceClass: !If [IsProduction, db.t3.medium, db.t3.micro]
      MultiAZ: !If [IsProduction, true, false]
```

### Pattern 3: Resource dependencies

```yaml
Resources:
  # Explicit dependency
  WebServer:
    Type: AWS::EC2::Instance
    DependsOn: DatabaseInstance
    Properties:
      # ...
  
  # Implicit dependency via Ref
  SecurityGroupIngress:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !Ref SecurityGroup
      SourceSecurityGroupId: !Ref LoadBalancerSecurityGroup
```

## Stack Organization Strategies

### Strategy 1: Layered architecture

```
Foundation Layer (rarely changes)
├── Network Stack (VPC, subnets, NAT)
└── Security Stack (IAM roles, KMS keys)

Platform Layer (occasional changes)
├── Database Stack (RDS, ElastiCache)
└── Storage Stack (S3, EFS)

Application Layer (frequent changes)
├── Compute Stack (EC2, ECS, Lambda)
└── API Stack (API Gateway, ALB)
```

### Strategy 2: Service-oriented

```
Per-service stacks:
├── User Service Stack
├── Order Service Stack
├── Payment Service Stack
└── Shared Infrastructure Stack
```

### Strategy 3: Environment isolation

```
Per-environment stacks:
├── Dev Environment
│   ├── Network
│   ├── Application
│   └── Data
├── Staging Environment
│   ├── Network
│   ├── Application
│   └── Data
└── Production Environment
    ├── Network
    ├── Application
    └── Data
```

## Advanced

For detailed information, see:
- [Nested Stacks](reference/nested-stacks.md) - Nested stack patterns and best practices
- [Parameters](reference/parameters.md) - Parameter design and validation strategies
- [Outputs](reference/outputs.md) - Output design and cross-stack references
