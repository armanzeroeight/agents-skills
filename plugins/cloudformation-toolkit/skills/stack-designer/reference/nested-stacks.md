# Nested Stacks

## Contents
- Nested stack basics
- Parent-child relationships
- Parameter passing
- Output retrieval
- Update strategies
- Best practices and patterns

## Nested Stack Basics

Nested stacks allow you to create reusable CloudFormation templates and organize complex infrastructure into manageable components.

### Why Use Nested Stacks

**Benefits:**
- Reuse common patterns across multiple stacks
- Overcome 51,200 byte template size limit
- Organize complex infrastructure logically
- Update components independently
- Share templates across teams

**When to use:**
- Template exceeds size limits
- Common patterns used multiple times
- Logical separation of concerns
- Team ownership boundaries
- Independent update cycles

### Basic Nested Stack

**Parent template:**
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Parent stack with nested stacks

Resources:
  NetworkStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/my-bucket/templates/network.yaml
      Parameters:
        EnvironmentName: !Ref EnvironmentName
      TimeoutInMinutes: 30
      Tags:
        - Key: Name
          Value: Network Infrastructure
```

**Child template (network.yaml):**
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Network infrastructure

Parameters:
  EnvironmentName:
    Type: String

Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      Tags:
        - Key: Name
          Value: !Sub '${EnvironmentName}-VPC'

Outputs:
  VpcId:
    Value: !Ref VPC
  VpcCidr:
    Value: !GetAtt VPC.CidrBlock
```

## Parent-Child Relationships

### Accessing Child Stack Outputs

**In parent template:**
```yaml
Resources:
  NetworkStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/my-bucket/network.yaml
  
  ApplicationStack:
    Type: AWS::CloudFormation::Stack
    DependsOn: NetworkStack
    Properties:
      TemplateURL: https://s3.amazonaws.com/my-bucket/application.yaml
      Parameters:
        # Access nested stack outputs
        VpcId: !GetAtt NetworkStack.Outputs.VpcId
        SubnetIds: !GetAtt NetworkStack.Outputs.PrivateSubnetIds
```

### Explicit Dependencies

```yaml
Resources:
  DatabaseStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/my-bucket/database.yaml
  
  ApplicationStack:
    Type: AWS::CloudFormation::Stack
    # Ensure database is created first
    DependsOn: DatabaseStack
    Properties:
      TemplateURL: https://s3.amazonaws.com/my-bucket/application.yaml
      Parameters:
        DatabaseEndpoint: !GetAtt DatabaseStack.Outputs.Endpoint
```

## Parameter Passing

### Simple Parameters

```yaml
Resources:
  ChildStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/my-bucket/child.yaml
      Parameters:
        # Pass parent parameters
        EnvironmentName: !Ref EnvironmentName
        
        # Pass static values
        InstanceType: t3.micro
        
        # Pass pseudo parameters
        Region: !Ref AWS::Region
        AccountId: !Ref AWS::AccountId
```

### Complex Parameters

```yaml
Resources:
  ChildStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/my-bucket/child.yaml
      Parameters:
        # Join list into comma-separated string
        SubnetIds: !Join [',', [!Ref Subnet1, !Ref Subnet2, !Ref Subnet3]]
        
        # Conditional parameter
        DatabaseSize: !If [IsProduction, 'large', 'small']
        
        # From mapping
        InstanceType: !FindInMap [EnvironmentMap, !Ref Environment, InstanceType]
```

### Parameter Validation in Child

**Child template:**
```yaml
Parameters:
  SubnetIds:
    Type: CommaDelimitedList
    Description: Comma-separated list of subnet IDs
  
  InstanceType:
    Type: String
    AllowedValues: [t3.micro, t3.small, t3.medium]
  
  DatabaseSize:
    Type: String
    AllowedValues: [small, medium, large]

Resources:
  # Use split list
  AutoScalingGroup:
    Type: AWS::AutoScaling::AutoScalingGroup
    Properties:
      VPCZoneIdentifier: !Ref SubnetIds
```

## Output Retrieval

### Accessing Nested Stack Outputs

```yaml
Resources:
  NetworkStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/my-bucket/network.yaml

Outputs:
  # Expose nested stack output
  VpcId:
    Description: VPC ID from nested stack
    Value: !GetAtt NetworkStack.Outputs.VpcId
  
  # Transform nested stack output
  VpcCidrBlock:
    Description: VPC CIDR block
    Value: !GetAtt NetworkStack.Outputs.VpcCidr
  
  # Combine multiple nested outputs
  AllSubnetIds:
    Description: All subnet IDs
    Value: !Join
      - ','
      - - !GetAtt NetworkStack.Outputs.PublicSubnet1
        - !GetAtt NetworkStack.Outputs.PublicSubnet2
        - !GetAtt NetworkStack.Outputs.PrivateSubnet1
        - !GetAtt NetworkStack.Outputs.PrivateSubnet2
```

### Chaining Nested Stacks

```yaml
Resources:
  # First nested stack
  NetworkStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/my-bucket/network.yaml
  
  # Second nested stack uses first's outputs
  SecurityStack:
    Type: AWS::CloudFormation::Stack
    DependsOn: NetworkStack
    Properties:
      TemplateURL: https://s3.amazonaws.com/my-bucket/security.yaml
      Parameters:
        VpcId: !GetAtt NetworkStack.Outputs.VpcId
  
  # Third nested stack uses both previous outputs
  ApplicationStack:
    Type: AWS::CloudFormation::Stack
    DependsOn: [NetworkStack, SecurityStack]
    Properties:
      TemplateURL: https://s3.amazonaws.com/my-bucket/application.yaml
      Parameters:
        VpcId: !GetAtt NetworkStack.Outputs.VpcId
        SecurityGroupId: !GetAtt SecurityStack.Outputs.SecurityGroupId
```

## Update Strategies

### Update Behavior

**Parent stack update:**
- Updates propagate to nested stacks
- Nested stacks update in dependency order
- Failed nested stack update fails parent

**Nested stack update:**
- Can update nested stack independently
- Parent stack shows nested stack in UPDATE_IN_PROGRESS
- Rollback affects only the nested stack

### Update Policies

```yaml
Resources:
  ApplicationStack:
    Type: AWS::CloudFormation::Stack
    UpdateReplacePolicy: Retain
    DeletionPolicy: Retain
    Properties:
      TemplateURL: https://s3.amazonaws.com/my-bucket/application.yaml
      # Prevent accidental updates
      NotificationARNs:
        - !Ref UpdateNotificationTopic
```

### Change Sets with Nested Stacks

```bash
# Create change set for parent stack
aws cloudformation create-change-set \
  --stack-name parent-stack \
  --change-set-name my-changes \
  --template-body file://parent.yaml \
  --include-nested-stacks

# Review changes (includes nested stack changes)
aws cloudformation describe-change-set \
  --stack-name parent-stack \
  --change-set-name my-changes

# Execute change set
aws cloudformation execute-change-set \
  --stack-name parent-stack \
  --change-set-name my-changes
```

## Best Practices

### Template Storage

**S3 bucket organization:**
```
s3://my-cfn-templates/
├── version-1.0/
│   ├── network.yaml
│   ├── security.yaml
│   └── application.yaml
├── version-1.1/
│   ├── network.yaml
│   ├── security.yaml
│   └── application.yaml
└── latest/
    ├── network.yaml
    ├── security.yaml
    └── application.yaml
```

**Versioned URLs:**
```yaml
Resources:
  NetworkStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      # Use versioned URL
      TemplateURL: https://s3.amazonaws.com/my-bucket/version-1.0/network.yaml
      # Or use version ID
      # TemplateURL: https://s3.amazonaws.com/my-bucket/network.yaml?versionId=abc123
```

### Error Handling

**Timeout configuration:**
```yaml
Resources:
  LongRunningStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/my-bucket/database.yaml
      # Increase timeout for slow resources
      TimeoutInMinutes: 60
```

**Notification configuration:**
```yaml
Resources:
  CriticalStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/my-bucket/critical.yaml
      NotificationARNs:
        - !Ref StackNotificationTopic
```

### Tagging Strategy

```yaml
Resources:
  NetworkStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/my-bucket/network.yaml
      Tags:
        - Key: Environment
          Value: !Ref EnvironmentName
        - Key: Owner
          Value: NetworkTeam
        - Key: CostCenter
          Value: Infrastructure
        - Key: ManagedBy
          Value: CloudFormation
```

## Common Patterns

### Pattern 1: Reusable VPC Template

**Parent stack:**
```yaml
Resources:
  DevVPC:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/my-bucket/vpc.yaml
      Parameters:
        EnvironmentName: dev
        VpcCidr: 10.0.0.0/16
  
  StagingVPC:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/my-bucket/vpc.yaml
      Parameters:
        EnvironmentName: staging
        VpcCidr: 10.1.0.0/16
  
  ProdVPC:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/my-bucket/vpc.yaml
      Parameters:
        EnvironmentName: prod
        VpcCidr: 10.2.0.0/16
```

### Pattern 2: Multi-Region Deployment

**Parent stack:**
```yaml
Resources:
  USEast1Stack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/my-bucket/regional.yaml
      Parameters:
        Region: us-east-1
        AvailabilityZones: us-east-1a,us-east-1b,us-east-1c
  
  USWest2Stack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/my-bucket/regional.yaml
      Parameters:
        Region: us-west-2
        AvailabilityZones: us-west-2a,us-west-2b,us-west-2c
```

### Pattern 3: Microservices Architecture

**Parent stack:**
```yaml
Resources:
  SharedInfrastructure:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/my-bucket/shared.yaml
  
  UserService:
    Type: AWS::CloudFormation::Stack
    DependsOn: SharedInfrastructure
    Properties:
      TemplateURL: https://s3.amazonaws.com/my-bucket/service.yaml
      Parameters:
        ServiceName: user-service
        VpcId: !GetAtt SharedInfrastructure.Outputs.VpcId
        SubnetIds: !GetAtt SharedInfrastructure.Outputs.PrivateSubnetIds
  
  OrderService:
    Type: AWS::CloudFormation::Stack
    DependsOn: SharedInfrastructure
    Properties:
      TemplateURL: https://s3.amazonaws.com/my-bucket/service.yaml
      Parameters:
        ServiceName: order-service
        VpcId: !GetAtt SharedInfrastructure.Outputs.VpcId
        SubnetIds: !GetAtt SharedInfrastructure.Outputs.PrivateSubnetIds
```

### Pattern 4: Environment Promotion

**Development:**
```yaml
# dev-stack.yaml
Resources:
  Infrastructure:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/my-bucket/version-1.0/infrastructure.yaml
      Parameters:
        Environment: dev
```

**Staging (after dev validation):**
```yaml
# staging-stack.yaml
Resources:
  Infrastructure:
    Type: AWS::CloudFormation::Stack
    Properties:
      # Same version as dev
      TemplateURL: https://s3.amazonaws.com/my-bucket/version-1.0/infrastructure.yaml
      Parameters:
        Environment: staging
```

**Production (after staging validation):**
```yaml
# prod-stack.yaml
Resources:
  Infrastructure:
    Type: AWS::CloudFormation::Stack
    Properties:
      # Same version as staging
      TemplateURL: https://s3.amazonaws.com/my-bucket/version-1.0/infrastructure.yaml
      Parameters:
        Environment: prod
```

## Troubleshooting

### Common Issues

**Issue: Nested stack update fails**
```
Error: Nested stack failed to update
```

**Solution:**
1. Check nested stack events in CloudFormation console
2. Review nested stack parameters
3. Verify template URL is accessible
4. Check IAM permissions for nested stack resources

**Issue: Circular dependency**
```
Error: Circular dependency between resources
```

**Solution:**
1. Review DependsOn relationships
2. Check parameter passing between stacks
3. Ensure outputs don't reference inputs from dependent stacks

**Issue: Template URL not accessible**
```
Error: S3 error: Access Denied
```

**Solution:**
1. Verify S3 bucket permissions
2. Check bucket policy allows CloudFormation access
3. Ensure template URL uses HTTPS
4. Verify S3 bucket is in same region

### Debugging Nested Stacks

**View nested stack events:**
```bash
# Get nested stack ID
aws cloudformation describe-stack-resources \
  --stack-name parent-stack \
  --logical-resource-id NetworkStack

# View nested stack events
aws cloudformation describe-stack-events \
  --stack-name <nested-stack-id>
```

**Check nested stack outputs:**
```bash
aws cloudformation describe-stacks \
  --stack-name <nested-stack-id> \
  --query 'Stacks[0].Outputs'
```

## Limits and Considerations

**CloudFormation limits:**
- Maximum 200 nested stacks per parent
- Maximum 5 levels of nesting
- Template size: 51,200 bytes (local), 460,800 bytes (S3)
- Maximum 200 parameters per template
- Maximum 200 outputs per template

**Performance considerations:**
- Nested stacks add deployment time
- Each nested stack is a separate API call
- Consider stack count vs. template size tradeoff
- Use parallel nested stacks when possible (no dependencies)

**Cost considerations:**
- No additional cost for nested stacks
- S3 storage costs for templates
- S3 request costs for template retrieval
