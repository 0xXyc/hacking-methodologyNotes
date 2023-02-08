---
description: The mystical cloud...
---

# ☁ Cloud Hacking

## Targeting AWS S3 Servers

### Methodology

1. Perform DNS reconnaissance on the target and attempt to grab an IP address
   1. host s3.amazonaws.com
   2. dnslookup \<ip\_here>
   3. dnsrecon s3.amazonaws.com
2. Install the aws binary if you need to:

{% embed url="https://docs.aws.amazon.com/cli/v1/userguide/install-macos.html#install-macosos-bundled-sudo" %}

3. Test the bucket's security:

{% embed url="https://bluexp.netapp.com/blog/aws-cvo-blg-amazon-s3-buckets-finding-open-buckets-with-grayhat-warfare" %}

* An S3 bucket can be accessed through one of the two link options:
* [http://s3.amazonaws.com/\[bucket\_name\]/](http://s3.amazonaws.com/\[bucket\_name]/)[http://\[bucket\_name\].s3.amazonaws.com/](https://community.rapid7.com/\[bucket\_name].s3.amazonaws.com)
