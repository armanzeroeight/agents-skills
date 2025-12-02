# Outputs

## Contents
- Output basics
- Export and import
- Cross-stack references
- Output design patterns
- Best practices
- Troubleshooting

## Output Basics

Outputs allow you to export values from your stack for use in other stacks or for display to users.

### Simple Output

```yaml
Resources:
  MyBucket:
    Type: AWS::S3::Bucket

Outputs:
  BucketName:
    Description: Name of the S3 bucket
    Value: !Ref MyBucket
```

### Output with Export

```yaml
Outputs:
  VpcId:
    Description: VPC ID
    Value: !Ref VPC
    Export:
      Name: !Sub '${AWS::StackName}-VpcId'
```

### Complete Output Structure

```yaml
Outputs:
  OutputName:
    Description: Human-readable description
    Value: The value to output
    Export:
      Name: Export name for cross-stack references
    Condition: Optional condition
```

## Export and Import

### Exporting Values

**Basic export:**
```yaml
Outputs:
  SecurityGroupId:
    Description: Security group ID
    Value: !Ref WebSecurityGroup
    Export:
      Name: WebSecurityGroupId
```

**Dynamic export name:**
```yaml
Outputs:
  SecurityGroupId:
    Description: Security group ID
    Value: !Ref WebSecurityGroup
    Export:
      Name: !Sub '${AWS::StackName}-SecurityGroupId'
```

**Export with environment:**
```yaml
Parameters:
  Environment:
    Type: String

Outputs:
  VpcId:
    Description: VPC ID
    Value: !Ref VPC
    Export:
      Name: !Sub '${Environment}-VpcId'
```

### Importing Values

**Basic import:**
```yaml
Resources:
  Instance:
    Type: AWS::EC2::Instance
    Properties:
      SecurityGroupIds:
        - !ImportValue WebSecurityGroupId
```

**Import with Sub:**
```yaml
Parameters:
  NetworkStackName:
    Type: String

Resources:
  Instance:
    Type: AWS::EC2::Instance
    Properties:
      SubnetId: !ImportValue
        Fn::Sub: '${NetworkStackName}-PublicSubnet1'
```

## Cross-Stack References

### Pattern 1: Network Stack → Application Stack

**Network stack (exports):**
```yaml
Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
  
  PublicSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.1.0/24
  
  PublicSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.2.0/24

Outputs:
  VpcId:
    Description: VPC ID
    Value: !Ref VPC
    Export:
      Name: !Sub '${AWS::StackName}-VpcId'
  
  PublicSubnetIds:
    Description: Public subnet IDs
    Value: !Join [',', [!Ref PublicSubnet1, !Ref PublicSubnet2]]
    Export:
      Name: !Sub '${AWS::StackName}-PublicSubnetIds'
```

**Application stack (imports):**
```yaml
Parameters:
  NetworkStackName:
    Type: String
    Default: NetworkStack

Resources:
  LoadBalancer:
    Type: AWS::ElasticLoadBalancingV2::LoadBalancer
    Properties:
      Subnets: !Split
        - ','
        - !ImportValue
          Fn::Sub: '${NetworkStackName}-PublicSubnetIds'
  
  SecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      VpcId: !ImportValue
        Fn::Sub: '${NetworkStackName}-VpcId'
```

### Pattern 2: Database Stack → Application Stack

**Database stack:**
```yaml
Resources:
  Database:
    Type: AWS::RDS::DBInstance
    Properties:
      DBInstanceClass: db.t3.micro
      Engine: postgres

Outputs:
  DatabaseEndpoint:
    Description: Database endpoint
    Value: !GetAtt Database.Endpoint.Address
    Export:
      Name: !Sub '${AWS::StackName}-Endpoint'
  
  DatabasePort:
    Description: Database port
    Value: !GetAtt Database.Endpoint.Port
    Export:
      Name: !Sub '${AWS::StackName}-Port'
```

**Application stack:**
```yaml
Parameters:
  DatabaseStackName:
    Type: String

Resources:
  TaskDefinition:
    Type: AWS::ECS::TaskDefinition
    Properties:
      ContainerDefinitions:
        - Name: app
          Environment:
            - Name: DB_HOST
              Value: !ImportValue
                Fn::Sub: '${DatabaseStackName}-Endpoint'
            - Name: DB_PORT
              Value: !ImportValue
                Fn::Sub: '${DatabaseStackName}-Port'
```

## Output Value Types

### Resource References

```yaml
Outputs:
  # Resource ID
  BucketName:
    Value: !Ref MyBucket
  
  # Resource attribute
  BucketArn:
    Value: !GetAtt MyBucket.Arn
  
  # Resource domain name
  BucketDomainName:
    Value: !GetAtt MyBucket.DomainName
```

### Computed Values

```yaml
Outputs:
  # Joined list
  AllSubnetIds:
    Value: !Join
      - ','
      - - !Ref Subnet1
        - !Ref Subnet2
        - !Ref Subnet3
  
  # Substituted string
  BucketUrl:
    Value: !Sub 'https://${MyBucket}.s3.amazonaws.com'
  
  # Selected value
  InstanceType:
    Value: !FindInMap [EnvironmentMap, !Ref Environment, InstanceType]
```

### Conditional Outputs

```yaml
Conditions:
  CreateDatabase: !Equals [!Ref CreateDB, 'true']

Outputs:
  DatabaseEndpoint:
    Condition: CreateDatabase
    Description: Database endpoint (only if created)
    Value: !GetAtt Database.Endpoint.Address
```

## Output Design Patterns

### Pattern 1: Comprehensive Network Outputs

```yaml
Outputs:
  # VPC
  VpcId:
    Description: VPC ID
    Value: !Ref VPC
    Export:
      Name: !Sub '${AWS::StackName}-VpcId'
  
  VpcCidr:
    Description: VPC CIDR block
    Value: !GetAtt VPC.CidrBlock
    Export:
      Name: !Sub '${AWS::StackName}-VpcCidr'
  
  # Subnets
  PublicSubnetIds:
    Description: Public subnet IDs (comma-separated)
    Value: !Join [',', [!Ref PublicSubnet1, !Ref PublicSubnet2]]
    Export:
      Name: !Sub '${AWS::StackName}-PublicSubnetIds'
  
  PrivateSubnetIds:
    Description: Private subnet IDs (comma-separated)
    Value: !Join [',', [!Ref PrivateSubnet1, !Ref PrivateSubnet2]]
    Export:
      Name: !Sub '${AWS::StackName}-PrivateSubnetIds'
  
  # Availability Zones
  AvailabilityZones:
    Description: Availability zones used
    Value: !Join [',', [!GetAtt PublicSubnet1.AvailabilityZone, !GetAtt PublicSubnet2.AvailabilityZone]]
```

### Pattern 2: Security Group Outputs

```yaml
Outputs:
  WebSecurityGroupId:
    Description: Web tier security group
    Value: !Ref WebSecurityGroup
    Export:
      Name: !Sub '${AWS::StackName}-WebSG'
  
  AppSecurityGroupId:
    Description: Application tier security group
    Value: !Ref AppSecurityGroup
    Export:
      Name: !Sub '${AWS::StackName}-AppSG'
  
  DatabaseSecurityGroupId:
    Description: Database tier security group
    Value: !Ref DatabaseSecurityGroup
    Export:
      Name: !Sub '${AWS::StackName}-DatabaseSG'
```

### Pattern 3: Load Balancer Outputs

```yaml
Outputs:
  LoadBalancerArn:
    Description: Load balancer ARN
    Value: !Ref LoadBalancer
    Export:
      Name: !Sub '${AWS::StackName}-ALB-Arn'
  
  LoadBalancerDNS:
    Description: Load balancer DNS name
    Value: !GetAtt LoadBalancer.DNSName
    Export:
      Name: !Sub '${AWS::StackName}-ALB-DNS'
  
  LoadBalancerUrl:
    Description: Load balancer URL
    Value: !Sub 'https://${LoadBalancer.DNSName}'
  
  TargetGroupArn:
    Description: Target group ARN
    Value: !Ref TargetGroup
    Export:
      Name: !Sub '${AWS::StackName}-TargetGroup'
```

### Pattern 4: IAM Role Outputs

```yaml
Outputs:
  TaskRoleArn:
    Description: ECS task role ARN
    Value: !GetAtt TaskRole.Arn
    Export:
      Name: !Sub '${AWS::StackName}-TaskRole'
  
  ExecutionRoleArn:
    Description: ECS execution role ARN
    Value: !GetAtt ExecutionRole.Arn
    Export:
      Name: !Sub '${AWS::StackName}-ExecutionRole'
```

## Best Practices

### Naming Conventions

**Use stack name prefix:**
```yaml
Outputs:
  VpcId:
    Value: !Ref VPC
    Export:
      Name: !Sub '${AWS::StackName}-VpcId'
```

**Include environment:**
```yaml
Parameters:
  Environment:
    Type: String

Outputs:
  VpcId:
    Value: !Ref VPC
    Export:
      Name: !Sub '${Environment}-${AWS::StackName}-VpcId'
```

**Use descriptive names:**
```yaml
# Good
Export:
  Name: !Sub '${AWS::StackName}-PublicSubnetIds'

# Avoid
Export:
  Name: !Sub '${AWS::StackName}-Subnets'  # Which subnets?
```

### Description Guidelines

**Be specific:**
```yaml
Outputs:
  VpcId:
    Description: VPC ID for application resources
    Value: !Ref VPC
```

**Include usage context:**
```yaml
Outputs:
  SecurityGroupId:
    Description: Security group for web tier (allows HTTP/HTTPS)
    Value: !Ref WebSecurityGroup
```

**Document format for complex values:**
```yaml
Outputs:
  SubnetIds:
    Description: Comma-separated list of private subnet IDs
    Value: !Join [',', [!Ref Subnet1, !Ref Subnet2]]
```

### Export Strategy

**Export what others need:**
```yaml
# Export
Outputs:
  VpcId:
    Value: !Ref VPC
    Export:
      Name: !Sub '${AWS::StackName}-VpcId'

# Don't export internal details
Outputs:
  InternalTableName:
    Value: !Ref InternalTable
    # No export - internal use only
```

**Version exports for breaking changes:**
```yaml
Outputs:
  ApiEndpointV2:
    Description: API endpoint (v2)
    Value: !Sub 'https://${ApiGateway}.execute-api.${AWS::Region}.amazonaws.com/v2'
    Export:
      Name: !Sub '${AWS::StackName}-ApiEndpoint-v2'
```

### Conditional Outputs

```yaml
Conditions:
  CreateDatabase: !Equals [!Ref CreateDB, 'true']
  IsProduction: !Equals [!Ref Environment, 'prod']

Outputs:
  DatabaseEndpoint:
    Condition: CreateDatabase
    Description: Database endpoint
    Value: !GetAtt Database.Endpoint.Address
    Export:
      Name: !Sub '${AWS::StackName}-DatabaseEndpoint'
  
  BackupVaultArn:
    Condition: IsProduction
    Description: Backup vault ARN (production only)
    Value: !GetAtt BackupVault.Arn
```

## Advanced Patterns

### Pattern: Multi-Value Outputs

**Export multiple related values:**
```yaml
Outputs:
  # Individual values
  PublicSubnet1:
    Value: !Ref PublicSubnet1
    Export:
      Name: !Sub '${AWS::StackName}-PublicSubnet1'
  
  PublicSubnet2:
    Value: !Ref PublicSubnet2
    Export:
      Name: !Sub '${AWS::StackName}-PublicSubnet2'
  
  # Combined list
  PublicSubnetIds:
    Value: !Join [',', [!Ref PublicSubnet1, !Ref PublicSubnet2]]
    Export:
      Name: !Sub '${AWS::StackName}-PublicSubnetIds'
```

**Import and split:**
```yaml
Resources:
  LoadBalancer:
    Type: AWS::ElasticLoadBalancingV2::LoadBalancer
    Properties:
      Subnets: !Split
        - ','
        - !ImportValue NetworkStack-PublicSubnetIds
```

### Pattern: Nested Stack Outputs

**Parent stack exposes nested outputs:**
```yaml
Resources:
  NetworkStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/bucket/network.yaml

Outputs:
  # Expose nested stack output
  VpcId:
    Description: VPC ID from nested stack
    Value: !GetAtt NetworkStack.Outputs.VpcId
    Export:
      Name: !Sub '${AWS::StackName}-VpcId'
  
  # Transform nested output
  VpcCidrBlock:
    Description: VPC CIDR block
    Value: !GetAtt NetworkStack.Outputs.VpcCidr
```

### Pattern: Computed Outputs

```yaml
Outputs:
  # URL construction
  ApplicationUrl:
    Description: Application URL
    Value: !Sub 'https://${LoadBalancer.DNSName}/app'
  
  # ARN construction
  LogGroupArn:
    Description: CloudWatch log group ARN
    Value: !Sub 'arn:aws:logs:${AWS::Region}:${AWS::AccountId}:log-group:${LogGroup}'
  
  # Conditional URL
  ApiEndpoint:
    Description: API endpoint
    Value: !If
      - UseCustomDomain
      - !Sub 'https://${CustomDomain}/api'
      - !Sub 'https://${ApiGateway}.execute-api.${AWS::Region}.amazonaws.com/prod'
```

## Troubleshooting

### Common Issues

**Issue: Export name already exists**
```
Error: Export <name> already exists
```

**Solution:**
- Use unique export names with stack name prefix
- Check for conflicting exports in other stacks
- Delete or rename conflicting export

**Issue: Cannot delete stack with exported outputs**
```
Error: Export <name> is still imported by other stacks
```

**Solution:**
1. Find importing stacks:
```bash
aws cloudformation list-imports --export-name MyExport
```

2. Update or delete importing stacks first
3. Then delete exporting stack

**Issue: Import value not found**
```
Error: No export named <name> found
```

**Solution:**
- Verify export name spelling
- Check exporting stack exists
- Ensure exporting stack is in same region
- Verify export is not conditional (and condition is true)

### Debugging Outputs

**List stack outputs:**
```bash
aws cloudformation describe-stacks \
  --stack-name MyStack \
  --query 'Stacks[0].Outputs'
```

**List all exports:**
```bash
aws cloudformation list-exports
```

**Find stacks importing an export:**
```bash
aws cloudformation list-imports \
  --export-name MyExport
```

**View export details:**
```bash
aws cloudformation list-exports \
  --query 'Exports[?Name==`MyExport`]'
```

## Limits and Considerations

**CloudFormation output limits:**
- Maximum 200 outputs per template
- Output name: 255 characters max
- Output value: 4096 characters max
- Description: 1024 characters max
- Export name: 255 characters max

**Export considerations:**
- Export names must be unique within region
- Cannot delete stack with exports being imported
- Cannot update export name (must delete and recreate)
- Exports are region-specific

**Performance considerations:**
- Outputs add minimal overhead
- Exports enable loose coupling between stacks
- Consider using SSM Parameter Store for dynamic values
- Use nested stacks for tight coupling

## Migration Strategies

### Migrating from Hardcoded Values to Outputs

**Before (hardcoded):**
```yaml
# Stack A
Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16

# Stack B (hardcoded VPC ID)
Resources:
  SecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      VpcId: vpc-12345678  # Hardcoded!
```

**After (using outputs):**
```yaml
# Stack A
Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16

Outputs:
  VpcId:
    Value: !Ref VPC
    Export:
      Name: NetworkStack-VpcId

# Stack B (using import)
Resources:
  SecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      VpcId: !ImportValue NetworkStack-VpcId
```

### Migrating from Parameters to Outputs

**Before (passing via parameters):**
```yaml
# Deploy Stack A, note VPC ID
# Deploy Stack B with VPC ID as parameter

# Stack B
Parameters:
  VpcId:
    Type: AWS::EC2::VPC::Id

Resources:
  SecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      VpcId: !Ref VpcId
```

**After (using outputs):**
```yaml
# Stack A exports VPC ID
# Stack B imports VPC ID

# Stack B
Resources:
  SecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      VpcId: !ImportValue NetworkStack-VpcId
```

**Benefits:**
- No manual parameter passing
- Automatic updates when source changes
- Type safety
- Dependency tracking
