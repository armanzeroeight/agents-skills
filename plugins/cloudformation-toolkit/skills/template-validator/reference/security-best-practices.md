# Security Best Practices

## Contents
- IAM security
- Network security
- Data encryption
- Secrets management
- Resource access control
- Compliance and auditing

## IAM Security

### Least Privilege Principle

**Bad - Overly permissive:**
```yaml
Resources:
  AppRole:
    Type: AWS::IAM::Role
    Properties:
      Policies:
        - PolicyName: AppPolicy
          PolicyDocument:
            Statement:
              - Effect: Allow
                Action: '*'  # All actions!
                Resource: '*'  # All resources!
```

**Good - Specific permissions:**
```yaml
Resources:
  AppRole:
    Type: AWS::IAM::Role
    Properties:
      Policies:
        - PolicyName: AppPolicy
          PolicyDocument:
            Statement:
              - Effect: Allow
                Action:
                  - s3:GetObject
                  - s3:PutObject
                Resource: !Sub '${DataBucket.Arn}/*'
              
              - Effect: Allow
                Action:
                  - dynamodb:GetItem
                  - dynamodb:PutItem
                  - dynamodb:Query
                Resource: !GetAtt DataTable.Arn
```

### Avoid Wildcard Actions

**Bad:**
```yaml
Action:
  - s3:*  # All S3 actions
  - ec2:*  # All EC2 actions
```

**Good:**
```yaml
Action:
  - s3:GetObject
  - s3:PutObject
  - s3:ListBucket
  - ec2:DescribeInstances
  - ec2:StartInstances
  - ec2:StopInstances
```

### Restrict Resource Access

**Bad:**
```yaml
Resource: '*'  # All resources
```

**Good:**
```yaml
Resource:
  - !Sub '${MyBucket.Arn}'
  - !Sub '${MyBucket.Arn}/*'
  - !GetAtt MyTable.Arn
```

### Assume Role Policies

**Restrict who can assume:**
```yaml
Resources:
  LambdaRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Statement:
          - Effect: Allow
            Principal:
              Service: lambda.amazonaws.com
            Action: sts:AssumeRole
            Condition:
              StringEquals:
                'sts:ExternalId': !Ref ExternalId
```

### Service Control Policies

**Prevent privilege escalation:**
```yaml
Statement:
  - Effect: Deny
    Action:
      - iam:CreateUser
      - iam:CreateRole
      - iam:AttachUserPolicy
      - iam:AttachRolePolicy
      - iam:PutUserPolicy
      - iam:PutRolePolicy
    Resource: '*'
    Condition:
      StringNotEquals:
        'aws:PrincipalArn': !Sub 'arn:aws:iam::${AWS::AccountId}:role/AdminRole'
```

## Network Security

### Security Group Rules

**Bad - Open to world:**
```yaml
Resources:
  WebSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      SecurityGroupIngress:
        # SSH open to world
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
        
        # Database open to world
        - IpProtocol: tcp
          FromPort: 3306
          ToPort: 3306
          CidrIp: 0.0.0.0/0
```

**Good - Restricted access:**
```yaml
Resources:
  BastionSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      SecurityGroupIngress:
        # SSH only from corporate network
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 203.0.113.0/24
          Description: Corporate network
  
  WebSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      SecurityGroupIngress:
        # HTTP/HTTPS from anywhere (public web)
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 443
          ToPort: 443
          CidrIp: 0.0.0.0/0
  
  DatabaseSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      SecurityGroupIngress:
        # Database only from application tier
        - IpProtocol: tcp
          FromPort: 3306
          ToPort: 3306
          SourceSecurityGroupId: !Ref AppSecurityGroup
```

### Network ACLs

**Restrict subnet access:**
```yaml
Resources:
  PrivateNetworkAcl:
    Type: AWS::EC2::NetworkAcl
    Properties:
      VpcId: !Ref VPC
  
  InboundRule:
    Type: AWS::EC2::NetworkAclEntry
    Properties:
      NetworkAclId: !Ref PrivateNetworkAcl
      RuleNumber: 100
      Protocol: 6  # TCP
      RuleAction: allow
      CidrBlock: 10.0.0.0/16  # Only from VPC
      PortRange:
        From: 443
        To: 443
```

### VPC Endpoints

**Use VPC endpoints for AWS services:**
```yaml
Resources:
  S3Endpoint:
    Type: AWS::EC2::VPCEndpoint
    Properties:
      VpcId: !Ref VPC
      ServiceName: !Sub 'com.amazonaws.${AWS::Region}.s3'
      RouteTableIds:
        - !Ref PrivateRouteTable
  
  DynamoDBEndpoint:
    Type: AWS::EC2::VPCEndpoint
    Properties:
      VpcId: !Ref VPC
      ServiceName: !Sub 'com.amazonaws.${AWS::Region}.dynamodb'
      RouteTableIds:
        - !Ref PrivateRouteTable
```

## Data Encryption

### S3 Bucket Encryption

**Enable encryption:**
```yaml
Resources:
  DataBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketEncryption:
        ServerSideEncryptionConfiguration:
          - ServerSideEncryptionByDefault:
              SSEAlgorithm: AES256
      
      # Or use KMS
      # BucketEncryption:
      #   ServerSideEncryptionConfiguration:
      #     - ServerSideEncryptionByDefault:
      #         SSEAlgorithm: aws:kms
      #         KMSMasterKeyID: !Ref EncryptionKey
      
      PublicAccessBlockConfiguration:
        BlockPublicAcls: true
        BlockPublicPolicy: true
        IgnorePublicAcls: true
        RestrictPublicBuckets: true
```

### EBS Volume Encryption

**Encrypt volumes:**
```yaml
Resources:
  LaunchTemplate:
    Type: AWS::EC2::LaunchTemplate
    Properties:
      LaunchTemplateData:
        BlockDeviceMappings:
          - DeviceName: /dev/xvda
            Ebs:
              VolumeSize: 20
              VolumeType: gp3
              Encrypted: true
              KmsKeyId: !Ref EncryptionKey
```

### RDS Encryption

**Encrypt database:**
```yaml
Resources:
  Database:
    Type: AWS::RDS::DBInstance
    Properties:
      StorageEncrypted: true
      KmsKeyId: !Ref EncryptionKey
      
      # Enable encryption in transit
      EnableIAMDatabaseAuthentication: true
```

### KMS Key Management

**Create and use KMS keys:**
```yaml
Resources:
  EncryptionKey:
    Type: AWS::KMS::Key
    Properties:
      Description: Encryption key for application data
      KeyPolicy:
        Statement:
          - Sid: Enable IAM policies
            Effect: Allow
            Principal:
              AWS: !Sub 'arn:aws:iam::${AWS::AccountId}:root'
            Action: 'kms:*'
            Resource: '*'
          
          - Sid: Allow service usage
            Effect: Allow
            Principal:
              Service:
                - s3.amazonaws.com
                - rds.amazonaws.com
            Action:
              - 'kms:Decrypt'
              - 'kms:GenerateDataKey'
            Resource: '*'
  
  KeyAlias:
    Type: AWS::KMS::Alias
    Properties:
      AliasName: alias/app-encryption-key
      TargetKeyId: !Ref EncryptionKey
```

## Secrets Management

### AWS Secrets Manager

**Store secrets securely:**
```yaml
Resources:
  DatabaseSecret:
    Type: AWS::SecretsManager::Secret
    Properties:
      Description: Database credentials
      GenerateSecretString:
        SecretStringTemplate: '{"username": "admin"}'
        GenerateStringKey: password
        PasswordLength: 32
        ExcludeCharacters: '"@/\'
        RequireEachIncludedType: true
  
  Database:
    Type: AWS::RDS::DBInstance
    Properties:
      MasterUsername: !Sub '{{resolve:secretsmanager:${DatabaseSecret}:SecretString:username}}'
      MasterUserPassword: !Sub '{{resolve:secretsmanager:${DatabaseSecret}:SecretString:password}}'
```

### SSM Parameter Store

**Store configuration securely:**
```yaml
Resources:
  ApiKey:
    Type: AWS::SSM::Parameter
    Properties:
      Name: /myapp/api-key
      Type: SecureString
      Value: !Ref ApiKeyValue
      Description: API key for external service
  
  Application:
    Type: AWS::Lambda::Function
    Properties:
      Environment:
        Variables:
          API_KEY: !Sub '{{resolve:ssm-secure:/myapp/api-key}}'
```

### Avoid Hardcoded Secrets

**Bad:**
```yaml
Resources:
  Database:
    Type: AWS::RDS::DBInstance
    Properties:
      MasterUserPassword: 'MyPassword123!'  # Never do this!
```

**Good:**
```yaml
Resources:
  Database:
    Type: AWS::RDS::DBInstance
    Properties:
      MasterUserPassword: !Sub '{{resolve:secretsmanager:${DatabaseSecret}:SecretString:password}}'
```

## Resource Access Control

### S3 Bucket Policies

**Restrict access:**
```yaml
Resources:
  DataBucket:
    Type: AWS::S3::Bucket
  
  BucketPolicy:
    Type: AWS::S3::BucketPolicy
    Properties:
      Bucket: !Ref DataBucket
      PolicyDocument:
        Statement:
          # Deny unencrypted uploads
          - Sid: DenyUnencryptedUploads
            Effect: Deny
            Principal: '*'
            Action: s3:PutObject
            Resource: !Sub '${DataBucket.Arn}/*'
            Condition:
              StringNotEquals:
                's3:x-amz-server-side-encryption': 'AES256'
          
          # Deny insecure transport
          - Sid: DenyInsecureTransport
            Effect: Deny
            Principal: '*'
            Action: 's3:*'
            Resource:
              - !GetAtt DataBucket.Arn
              - !Sub '${DataBucket.Arn}/*'
            Condition:
              Bool:
                'aws:SecureTransport': false
          
          # Allow specific role
          - Sid: AllowAppRole
            Effect: Allow
            Principal:
              AWS: !GetAtt AppRole.Arn
            Action:
              - s3:GetObject
              - s3:PutObject
            Resource: !Sub '${DataBucket.Arn}/*'
```

### Lambda Function Permissions

**Restrict invocation:**
```yaml
Resources:
  Function:
    Type: AWS::Lambda::Function
    Properties:
      # Function configuration
  
  FunctionPermission:
    Type: AWS::Lambda::Permission
    Properties:
      FunctionName: !Ref Function
      Action: lambda:InvokeFunction
      Principal: apigateway.amazonaws.com
      SourceArn: !Sub 'arn:aws:execute-api:${AWS::Region}:${AWS::AccountId}:${Api}/*'
```

### API Gateway Authorization

**Require authentication:**
```yaml
Resources:
  Api:
    Type: AWS::ApiGateway::RestApi
    Properties:
      Name: SecureAPI
  
  Authorizer:
    Type: AWS::ApiGateway::Authorizer
    Properties:
      Name: CognitoAuthorizer
      Type: COGNITO_USER_POOLS
      IdentitySource: method.request.header.Authorization
      RestApiId: !Ref Api
      ProviderARNs:
        - !GetAtt UserPool.Arn
  
  Method:
    Type: AWS::ApiGateway::Method
    Properties:
      RestApiId: !Ref Api
      ResourceId: !Ref Resource
      HttpMethod: GET
      AuthorizationType: COGNITO_USER_POOLS
      AuthorizerId: !Ref Authorizer
```

## Compliance and Auditing

### CloudTrail Logging

**Enable audit logging:**
```yaml
Resources:
  Trail:
    Type: AWS::CloudTrail::Trail
    Properties:
      S3BucketName: !Ref LogBucket
      IsLogging: true
      IsMultiRegionTrail: true
      IncludeGlobalServiceEvents: true
      EnableLogFileValidation: true
      EventSelectors:
        - ReadWriteType: All
          IncludeManagementEvents: true
          DataResources:
            - Type: AWS::S3::Object
              Values:
                - !Sub '${DataBucket.Arn}/*'
```

### Config Rules

**Monitor compliance:**
```yaml
Resources:
  EncryptionRule:
    Type: AWS::Config::ConfigRule
    Properties:
      ConfigRuleName: s3-bucket-encryption-enabled
      Source:
        Owner: AWS
        SourceIdentifier: S3_BUCKET_SERVER_SIDE_ENCRYPTION_ENABLED
  
  PublicAccessRule:
    Type: AWS::Config::ConfigRule
    Properties:
      ConfigRuleName: s3-bucket-public-read-prohibited
      Source:
        Owner: AWS
        SourceIdentifier: S3_BUCKET_PUBLIC_READ_PROHIBITED
```

### GuardDuty

**Enable threat detection:**
```yaml
Resources:
  Detector:
    Type: AWS::GuardDuty::Detector
    Properties:
      Enable: true
      FindingPublishingFrequency: FIFTEEN_MINUTES
```

### Security Hub

**Centralize security findings:**
```yaml
Resources:
  Hub:
    Type: AWS::SecurityHub::Hub
    Properties:
      Tags:
        Environment: !Ref Environment
```

## Security Checklist

**IAM:**
- [ ] Least privilege policies
- [ ] No wildcard actions
- [ ] Specific resources
- [ ] Assume role conditions
- [ ] No hardcoded credentials

**Network:**
- [ ] Security groups restrict access
- [ ] No 0.0.0.0/0 for sensitive ports
- [ ] VPC endpoints for AWS services
- [ ] Network ACLs configured
- [ ] Private subnets for databases

**Encryption:**
- [ ] S3 bucket encryption enabled
- [ ] EBS volumes encrypted
- [ ] RDS encryption enabled
- [ ] KMS keys for sensitive data
- [ ] Encryption in transit

**Secrets:**
- [ ] Secrets Manager for credentials
- [ ] SSM Parameter Store for config
- [ ] No hardcoded secrets
- [ ] Secrets rotation enabled
- [ ] NoEcho for sensitive parameters

**Access Control:**
- [ ] S3 bucket policies restrict access
- [ ] Lambda permissions specific
- [ ] API Gateway authorization
- [ ] Resource-based policies
- [ ] Public access blocked

**Auditing:**
- [ ] CloudTrail enabled
- [ ] Config rules configured
- [ ] GuardDuty enabled
- [ ] Security Hub enabled
- [ ] Log retention configured

## Common Security Issues

### Issue 1: Overly Permissive IAM

**Problem:**
```yaml
Action: '*'
Resource: '*'
```

**Fix:**
```yaml
Action:
  - s3:GetObject
  - s3:PutObject
Resource: !Sub '${MyBucket.Arn}/*'
```

### Issue 2: Open Security Groups

**Problem:**
```yaml
CidrIp: 0.0.0.0/0
FromPort: 22
```

**Fix:**
```yaml
CidrIp: 10.0.0.0/8
FromPort: 22
```

### Issue 3: Unencrypted Data

**Problem:**
```yaml
Type: AWS::S3::Bucket
# No encryption
```

**Fix:**
```yaml
Type: AWS::S3::Bucket
Properties:
  BucketEncryption:
    ServerSideEncryptionConfiguration:
      - ServerSideEncryptionByDefault:
          SSEAlgorithm: AES256
```

### Issue 4: Hardcoded Secrets

**Problem:**
```yaml
MasterUserPassword: 'password123'
```

**Fix:**
```yaml
MasterUserPassword: !Sub '{{resolve:secretsmanager:${Secret}:SecretString:password}}'
```

### Issue 5: Public S3 Buckets

**Problem:**
```yaml
Type: AWS::S3::Bucket
# No public access block
```

**Fix:**
```yaml
Type: AWS::S3::Bucket
Properties:
  PublicAccessBlockConfiguration:
    BlockPublicAcls: true
    BlockPublicPolicy: true
    IgnorePublicAcls: true
    RestrictPublicBuckets: true
```
