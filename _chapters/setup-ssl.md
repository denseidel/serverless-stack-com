---
layout: post
title: Set up SSL
date: 2017-02-11 00:00:00
lang: en
description: We want to enable SSL or HTTPS for our React.js app on AWS. To do so we are going to request a certificate using the Certificate Manager service from AWS. We are then going to use the new certificate in our CloudFront Distributions.
comments_id: comments-for-set-up-ssl/133
ref: setup-ssl
---

Now that we have our CloudFront distribution live, let's set up our domain with it. You can purchase a domain right from the [AWS Console](https://console.aws.amazon.com) by heading to the Route 53 section in the list of services.

![Select Route 53 service screenshot](/assets/select-route-53-service.png)

### Purchase a Domain with Route 53

Type in your domain in the **Register domain** section and click **Check**.

![Search available domain screenshot](/assets/search-available-domain.png)

After checking its availability, click **Add to cart**.

![Add domain to cart screenshot](/assets/add-domain-to-cart.png)

And hit **Continue** at the bottom of the page.

![Continue to contact details screenshot](/assets/continue-to-contact-detials.png)

Fill in your contact details and hit **Continue** once again.

![Continue to confirm details screenshot](/assets/continue-to-confirm-detials.png)

Finally, review your details and confirm the purchase by hitting **Complete Purchase**.

![Confirm domain purchase screenshot](/assets/confirm-domain-purchase.png)

Now that we have our domain, let's add a layer of security to it by switching to HTTPS. AWS makes this fairly easy to do, thanks to Certificate Manager. The SSL Certificate is [required](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/CNAMEs.html#alternate-domain-names-requirements) for configuring your domain as an alternative name on your CloudFront distribution.

### Request a Certificate

Select **Certificate Manager** from the list of services in your [AWS Console](https://console.aws.amazon.com). Ensure that you are in the **US East (N. Virginia)** region. This is because a certificate needs to be from this region for it to [work with CloudFront](http://docs.aws.amazon.com/acm/latest/userguide/acm-regions.html). 

![Select Certificate Manager service screenshot](/assets/select-certificate-manager-service.png)

If this is your first certificate, you'll need to hit **Get started**. If not then hit **Request a certificate** from the top.

![Get started with Certificate Manager screenshot](/assets/get-started-certificate-manager.png)

And type in the name of our domain. Hit **Add another name to this certificate** and add our www version of our domain as well. Hit **Review and request** once you are done.

![Add domain names to certificate screenshot](/assets/add-domain-names-to-certificate.png)

Now to confirm that we control the domain, select the **DNS validation** method and hit **Review**.

![Select dns validation for certificate screenshot](/assets/select-dns-validation-for-certificate.png)

On the validation screen expand the two domains we are trying to validate.

![Expand dns validation details screenshot](/assets/expand-dns-validation-details.png)

Since we control the domain through Route 53, we can directly create the DNS record through here by hitting **Create record in Route 53**.

![Create Route 53 dns record screenshot](/assets/create-route-53-dns-record.png)

And confirm that you want the record to be created by hitting **Create**.

![Confirm Route 53 dns record screenshot](/assets/confirm-route-53-dns-record.png)

Also, make sure to do this for the other domain.

The process of creating a DNS record and validating it can take around 30 minutes.

Next, we'll associate this certificate with our CloudFront Distributions.

You can do this step also with [cloudformation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-certificatemanager-certificate.html) yet this requires you to manually validate the domain ownership. Alternatively you can implement a [custom resource](https://binx.io/blog/2018/10/05/automated-provisioning-of-acm-certificates-using-route53-in-cloudformation/). For the sake o bootstraping we will do this step manually and verify the domain through email for the first time. Add the following to your could formation Stack


```yaml
  DomainCertificate:
    Type: AWS::CertificateManager::Certificate
    Properties: 
      DomainName: "d10l.de"
      SubjectAlternativeNames: ["www.d10l.de", "*.d10l.de"]
      ValidationMethod: DNS
```

After the stack has started check the log and create the DNS entry in Route53 to validate the domain.

Next, we'll add an alternate domain name for our CloudFront Distribution.

