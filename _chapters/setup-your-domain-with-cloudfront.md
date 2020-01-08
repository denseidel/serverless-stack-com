---
layout: post
title: Set up Your Domain with CloudFront
date: 2017-02-09 00:00:00
lang: en
description: To host our React.js app under our own domain name in AWS we are going to purchase a domain using Route 53. We will point the domain to our CloudFront Distribution with an Alias Resource Record Set. We also need to create an AAAA Record Set to support IPv6.
comments_id: set-up-your-domain-with-cloudfront/149
ref: set-up-your-domain-with-cloudfront
---

### Add Alternate Domain for CloudFront Distribution

Head over to the details of your CloudFront Distribution and hit **Edit**.

![Edit CloudFront Distribution screenshot](/assets/edit-cloudfront-distribution.png)

And type in your new domain name in the **Alternate Domain Names (CNAMEs)** field.

![Set alternate domain name screenshot](/assets/set-alternate-domain-name.png)

Scroll down and hit **Yes, Edit** to save the changes.

![Yes edit CloudFront changes screenshot](/assets/yes-edit-cloudfront-changes.png)

Next, let's point our domain to the CloudFront Distribution.


```yaml
  ProdCloudFrontDistribution:
    Type: 'AWS::CloudFront::Distribution'
    Properties:
      DistributionConfig:
        # Add the alternative domain here
        Aliases:
        - !Ref DomainName
        CustomErrorResponses:
        - ErrorCode: 404 # not found
          ResponseCode: 200
          ErrorCachingMinTTL: 300
          ResponsePagePath: !Sub '/${ErrorPagePath}' 
        DefaultCacheBehavior:
          AllowedMethods:
          - GET
          - HEAD
          - OPTIONS
          CachedMethods:
          - GET
          - HEAD
          - OPTIONS
          Compress: true
          DefaultTTL: 3600 # in seconds
          ForwardedValues:
            Cookies:
              Forward: none
            QueryString: false
          MaxTTL: 86400 # in seconds
          MinTTL: 60 # in seconds
          TargetOriginId: s3origin
          ViewerProtocolPolicy: 'allow-all'
        DefaultRootObject: !Ref DefaultRootObject
        Enabled: true
        HttpVersion: http2
        IPV6Enabled: true
        Origins:
        - DomainName: !GetAtt 'ProdLiveS3Bucket.DomainName'
          Id: s3origin
          S3OriginConfig:
            OriginAccessIdentity: !Sub 'origin-access-identity/cloudfront/${CloudFrontOriginAccessIdentity}'
        PriceClass: 'PriceClass_All'
        ViewerCertificate:
          AcmCertificateArn: !Ref DomainCertificate
          #CloudFrontDefaultCertificate: Boolean
          MinimumProtocolVersion: 'TLSv1.1_2016'
          SslSupportMethod: 'sni-only'
    ProdWWWCloudFrontDistribution:
    Type: 'AWS::CloudFront::Distribution'
    Properties:
      DistributionConfig:
        Aliases:
        - !Sub 'www.${DomainName}'
        Enabled: true
        HttpVersion: http2
        IPV6Enabled: true
        Origins:
        - DomainName: !GetAtt 'ProdLiveS3Bucket.DomainName'
          Id: www-s3origin
          S3OriginConfig:
            OriginAccessIdentity: !Sub 'origin-access-identity/cloudfront/${CloudFrontOriginAccessIdentity}'
        PriceClass: 'PriceClass_All'
        ViewerCertificate:
          AcmCertificateArn: !Ref DomainCertificate
          MinimumProtocolVersion: 'TLSv1.1_2016'
          SslSupportMethod: 'sni-only'
```

### Point Domain to CloudFront Distribution

Head back into Route 53 and hit the **Hosted Zones** button. If you don't have an existing **Hosted Zone**, you'll need to create one by adding the **Domain Name** and selecting **Public Hosted Zone** as the **Type**.

![Select Route 53 hosted zones screenshot](/assets/select-route-53-hosted-zones.png)

Select your domain from the list and hit **Create Record Set** in the details screen.

![Select create record set screenshot](/assets/select-create-record-set.png)

Leave the **Name** field empty since we are going to point our bare domain (without the www.) to our CloudFront Distribution.

![Leave name field empty screenshot](/assets/leave-name-field-empty.png)

And select **Alias** as **Yes** since we are going to simply point this to our CloudFront domain.

![Set Alias to yes screenshot](/assets/set-alias-to-yes.png)

In the **Alias Target** dropdown, select your CloudFront Distribution.

![Select your CloudFront Distribution screenshot](/assets/select-your-cloudfront-distribution.png)

Finally, hit **Create** to add your new record set.

![Select create to add record set screenshot](/assets/select-create-to-add-record-set.png)


```yaml
  ARecordSet:
    Type: AWS::Route53::RecordSet
    Properties:
      Name: !Ref DomainName
      AliasTarget: 
        DNSName: !GetAtt 'ProdCloudFrontDistribution.DomainName'
        HostedZoneId: 'Z2FDTNDATAQYW2'
      HostedZoneName: !Join ["", [!Ref DomainName, "."]]
      Type: A
      TTL: 60
  ARecordSetWWW:
    Type: AWS::Route53::RecordSet
    Properties:
      Name: !Sub 'www.${DomainName}'
      AliasTarget: 
        DNSName: !GetAtt 'ProdWWWCloudFrontDistribution.DomainName'
        HostedZoneId: 'Z2FDTNDATAQYW2'
      HostedZoneName: !Join ["", [!Ref DomainName, "."]]
      Type: A
      TTL: 60
```

### Add IPv6 Support

CloudFront Distributions have IPv6 enabled by default and this means that we need to create an AAAA record as well. It is set up exactly the same way as the Alias record.

Create a new Record Set with the exact settings as before, except make sure to pick **AAAA - IPv6 address** as the **Type**.

![Select AAAA IPv6 record set screenshot](/assets/select-create-aaaa-ipv6-record-set.png)

And hit **Create** to add your AAAA record set.

It can take around an hour to update the DNS records but once it's done, you should be able to access your app through your domain.

![App live on new domain screenshot](/assets/app-live-on-new-domain.png)

Next up, we'll take a quick look at ensuring that our www. domain also directs to our app.


Update the cloud formation template by:

- adding the alias to the cloudfront distribution
- adding paremeters for the hosted zone for the existing domain (alternatively if this was created by another stack this can be referenced)
- create A record set in the hosted zone for the domain / alias target is the cloudfront distribution
- create an AAAA record (IPv6 Support)


https://www.brautaset.org/articles/2017/route-53-cloudformation.html
https://docs.aws.amazon.com/de_de/AWSCloudFormation/latest/UserGuide/aws-properties-route53-recordset.html#cfn-route53-recordset-type

```yaml
  AAAARecordSet:
    Type: AWS::Route53::RecordSet
    Properties:
      Name: !Ref DomainName
      AliasTarget: 
        DNSName: !GetAtt 'ProdCloudFrontDistribution.DomainName'
        HostedZoneId: 'Z2FDTNDATAQYW2'
      HostedZoneName: !Join ["", [!Ref DomainName, "."]]
      Type: AAAA
      TTL: 60
  AAAARecordSetWWW:
    Type: AWS::Route53::RecordSet
    Properties:
      Name: !Sub 'www.${DomainName}'
      AliasTarget: 
        DNSName: !GetAtt 'ProdWWWCloudFrontDistribution.DomainName'
        HostedZoneId: 'Z2FDTNDATAQYW2'
      HostedZoneName: !Join ["", [!Ref DomainName, "."]]
      Type: AAAA
      TTL: 60
```

### Update CloudFront Distributions with Certificate

Open up our first CloudFront Distribution from our list of distributions and hit the **Edit** button.

![Select CloudFront Distribution screenshot](/assets/select-cloudfront-Distribution.png)

Now switch the **SSL Certificate** to **Custom SSL Certificate** and select the certificate we just created from the drop down. And scroll down to the bottom and hit **Yes, Edit**.

![Select custom SSL Certificate screenshot](/assets/select-custom-ssl-certificate.png)

Next, head over to the **Behaviors** tab from the top.

![Select Behaviors tab screenshot](/assets/select-behaviors-tab.png)

And select the only one we have and hit **Edit**.

![Edit Distribution Behavior screenshot](/assets/edit-distribution-behavior.png)

Then switch the **Viewer Protocol Policy** to **Redirect HTTP to HTTPS**. And scroll down to the bottom and hit **Yes, Edit**.

![Switch Viewer Protocol Policy screenshot](/assets/switch-viewer-protocol-policy.png)


```yaml
  ProdCloudFrontDistribution:
    Type: 'AWS::CloudFront::Distribution'
    Properties:
      DistributionConfig:
        Aliases:
        - !Ref DomainName
        CustomErrorResponses:
        - ErrorCode: 404 # not found
          ResponseCode: 200
          ErrorCachingMinTTL: 300
          ResponsePagePath: !Sub '/${ErrorPagePath}' 
        DefaultCacheBehavior:
          AllowedMethods:
          - GET
          - HEAD
          - OPTIONS
          CachedMethods:
          - GET
          - HEAD
          - OPTIONS
          Compress: true
          DefaultTTL: 3600 # in seconds
          ForwardedValues:
            Cookies:
              Forward: none
            QueryString: false
          MaxTTL: 86400 # in seconds
          MinTTL: 60 # in seconds
          TargetOriginId: s3origin
          #CHANCE THIS TO redirect-to-https 
          ViewerProtocolPolicy: 'redirect-to-https'
        DefaultRootObject: !Ref DefaultRootObject
        Enabled: true
        HttpVersion: http2
        IPV6Enabled: true
        Origins:
        - DomainName: !GetAtt 'ProdLiveS3Bucket.DomainName'
          Id: s3origin
          S3OriginConfig:
            OriginAccessIdentity: !Sub 'origin-access-identity/cloudfront/${CloudFrontOriginAccessIdentity}'
        PriceClass: 'PriceClass_All'
        # ADD THE CERTICIATE HERE
        ViewerCertificate:
          AcmCertificateArn: !Ref DomainCertificate
          #CloudFrontDefaultCertificate: Boolean
          MinimumProtocolVersion: 'TLSv1.1_2016'
          SslSupportMethod: 'sni-only'
```

Now let's do the same for our other CloudFront Distribution.

![Select custom SSL Certificate screenshot](/assets/select-custom-ssl-certificate-2.png)

But leave the **Viewer Protocol Policy** as **HTTP and HTTPS**. This is because we want our users to go straight to the HTTPS version of our non-www domain. As opposed to redirecting to the HTTPS version of our www domain before redirecting again.

![Dont switch Viewer Protocol Policy for www distribution screenshot](/assets/dont-switch-viewer-protocol-policy-for-www-distribution.png)

```yaml
  ProdWWWCloudFrontDistribution:
    Type: 'AWS::CloudFront::Distribution'
    Properties:
      DistributionConfig:
        Aliases:
        - !Sub 'www.${DomainName}'
        Enabled: true
        HttpVersion: http2
        IPV6Enabled: true
        Origins:
        - DomainName: !GetAtt 'ProdLiveS3Bucket.DomainName'
          Id: www-s3origin
          S3OriginConfig:
            OriginAccessIdentity: !Sub 'origin-access-identity/cloudfront/${CloudFrontOriginAccessIdentity}'
        PriceClass: 'PriceClass_All'
        ## Add the certificate
        ViewerCertificate:
          AcmCertificateArn: !Ref DomainCertificate
          MinimumProtocolVersion: 'TLSv1.1_2016'
          SslSupportMethod: 'sni-only'
```

### Update S3 Redirect Bucket

The S3 Redirect Bucket that we created in the last chapter is redirecting to the HTTP version of our non-www domain. We should switch this to the HTTPS version to prevent the extra redirect.

Open up the S3 Redirect Bucket we created in the last chapter. Head over to the **Properties** tab and select **Static website hosting**.

![Open S3 Redirect Bucket Properties screenshot](/assets/open-s3-redirect-bucket-properties.png)

Change the **Protocol** to **https** and hit **Save**.

![Change S3 Redirect to HTTPS screenshot](/assets/change-s3-redirect-to-https.png)

And that's it. Our app should be served out on our domain through HTTPS.

![App live with certificate screenshot](/assets/app-live-with-certificate.png)

Next up, let's look at the process of deploying updates to our app.

```yaml
  ProdLiveS3BucketWWWRedirect:
    Type: AWS::S3::Bucket
    Properties:
      BucketName:  !Sub 'www-${ApplicationName}-prod'
      WebsiteConfiguration:
        RedirectAllRequestsTo:
          HostName: !Ref DomainName
          Protocol: 'https'
```

Finally the complete template:

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: 'Static website hosting with S3 and CloudFront'
Parameters:
  DefaultRootObject:
    Description: 'The default path for the index document.'
    Type: String
    Default: 'index.html'
  ErrorPagePath:
    Description: 'The path of the error page for the website.'
    Type: String
    Default: 'index.html'
  ApplicationName:
    Description: 'The name of the application'
    Type: String
    Default: 'ssk-notes-app-client'
  DomainName:
    Type: String
    Default: d10l.de
Resources:
  # Create the bucket to contain the website HTML
  ProdLiveS3Bucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName:  !Sub '${ApplicationName}-prod'
      WebsiteConfiguration:
        IndexDocument: !Ref DefaultRootObject
        ErrorDocument: !Ref ErrorPagePath
  # Configure the bucket as a CloudFront Origin
  # Create the bucket redirect www traffic 
  ProdLiveS3BucketWWWRedirect:
    Type: AWS::S3::Bucket
    Properties:
      BucketName:  !Sub 'www-${ApplicationName}-prod'
      WebsiteConfiguration:
        RedirectAllRequestsTo:
          HostName: !Ref DomainName
          Protocol: 'https'
  ReadPolicy:
    Type: 'AWS::S3::BucketPolicy'
    Properties:
      Bucket: !Ref ProdLiveS3Bucket
      PolicyDocument:
        Statement:
        - Action: 's3:GetObject'
          Effect: Allow
          Resource: !Sub 'arn:aws:s3:::${ProdLiveS3Bucket}/*'
          Principal:
            CanonicalUser: !GetAtt CloudFrontOriginAccessIdentity.S3CanonicalUserId
  ReadPolicyWWW:
    Type: 'AWS::S3::BucketPolicy'
    Properties:
      Bucket: !Ref ProdLiveS3BucketWWWRedirect
      PolicyDocument:
        Statement:
        - Action: 's3:GetObject'
          Effect: Allow
          Resource: !Sub 'arn:aws:s3:::${ProdLiveS3BucketWWWRedirect}/*'
          Principal:
            CanonicalUser: !GetAtt CloudFrontOriginAccessIdentity.S3CanonicalUserId
  DomainCertificate:
    Type: AWS::CertificateManager::Certificate
    Properties: 
      DomainName: "d10l.de"
      SubjectAlternativeNames: ["www.d10l.de", "*.d10l.de"]
      ValidationMethod: DNS
  CloudFrontOriginAccessIdentity:
    Type: 'AWS::CloudFront::CloudFrontOriginAccessIdentity'
    Properties:
      CloudFrontOriginAccessIdentityConfig:
        Comment: !Ref ProdLiveS3Bucket
  ProdCloudFrontDistribution:
    Type: 'AWS::CloudFront::Distribution'
    Properties:
      DistributionConfig:
        Aliases:
        - !Ref DomainName
        CustomErrorResponses:
        - ErrorCode: 404 # not found
          ResponseCode: 200
          ErrorCachingMinTTL: 300
          ResponsePagePath: !Sub '/${ErrorPagePath}' 
        DefaultCacheBehavior:
          AllowedMethods:
          - GET
          - HEAD
          - OPTIONS
          CachedMethods:
          - GET
          - HEAD
          - OPTIONS
          Compress: true
          DefaultTTL: 3600 # in seconds
          ForwardedValues:
            Cookies:
              Forward: none
            QueryString: false
          MaxTTL: 86400 # in seconds
          MinTTL: 60 # in seconds
          TargetOriginId: s3origin
          ViewerProtocolPolicy: 'redirect-to-https'
        DefaultRootObject: !Ref DefaultRootObject
        Enabled: true
        HttpVersion: http2
        IPV6Enabled: true
        Origins:
        - DomainName: !GetAtt 'ProdLiveS3Bucket.DomainName'
          Id: s3origin
          S3OriginConfig:
            OriginAccessIdentity: !Sub 'origin-access-identity/cloudfront/${CloudFrontOriginAccessIdentity}'
        PriceClass: 'PriceClass_All'
        ViewerCertificate:
          AcmCertificateArn: !Ref DomainCertificate
          #CloudFrontDefaultCertificate: Boolean
          MinimumProtocolVersion: 'TLSv1.1_2016'
          SslSupportMethod: 'sni-only'
  ProdWWWCloudFrontDistribution:
    Type: 'AWS::CloudFront::Distribution'
    Properties:
      DistributionConfig:
        Aliases:
        - !Sub 'www.${DomainName}'
        Enabled: true
        HttpVersion: http2
        IPV6Enabled: true
        DefaultCacheBehavior:
          AllowedMethods:
          - GET
          - HEAD
          - OPTIONS
          CachedMethods:
          - GET
          - HEAD
          - OPTIONS
          Compress: true
          DefaultTTL: 3600 # in seconds
          ForwardedValues:
            Cookies:
              Forward: none
            QueryString: false
          MaxTTL: 86400 # in seconds
          MinTTL: 60 # in seconds
          TargetOriginId: www-s3origin
          ViewerProtocolPolicy: 'allow-all'
        Origins:
        - DomainName: !GetAtt 'ProdLiveS3Bucket.DomainName'
          Id: www-s3origin
          S3OriginConfig:
            OriginAccessIdentity: !Sub 'origin-access-identity/cloudfront/${CloudFrontOriginAccessIdentity}'
        PriceClass: 'PriceClass_All'
        ViewerCertificate:
          AcmCertificateArn: !Ref DomainCertificate
          MinimumProtocolVersion: 'TLSv1.1_2016'
          SslSupportMethod: 'sni-only'
  DNSRecords:
    Type: AWS::Route53::RecordSetGroup
    Properties:
      HostedZoneName: !Join ["", [!Ref DomainName, "."]]
      RecordSets: 
        - Name: !Ref DomainName
          AliasTarget: 
            DNSName: !GetAtt 'ProdCloudFrontDistribution.DomainName'
            HostedZoneId: 'Z2FDTNDATAQYW2'
          Type: A
        - Name: !Sub 'www.${DomainName}'
          AliasTarget: 
            DNSName: !GetAtt 'ProdWWWCloudFrontDistribution.DomainName'
            HostedZoneId: 'Z2FDTNDATAQYW2'
          Type: A
        - Name: !Ref DomainName
          AliasTarget: 
            DNSName: !GetAtt 'ProdCloudFrontDistribution.DomainName'
            HostedZoneId: 'Z2FDTNDATAQYW2'
          Type: AAAA
        - Name: !Sub 'www.${DomainName}'
          AliasTarget: 
            DNSName: !GetAtt 'ProdWWWCloudFrontDistribution.DomainName'
            HostedZoneId: 'Z2FDTNDATAQYW2'
          Type: AAAA
Outputs:
  BucketName:
    Description: 'S3 Bucket Name'
    Value: !Ref ProdLiveS3Bucket
  DistributionId:
    Description: 'CloudFront Distribution ID'
    Value: !Ref ProdCloudFrontDistribution
  Domain:
    Description: 'Cloudfront Domain'
    Value: !GetAtt ProdCloudFrontDistribution.DomainName
```