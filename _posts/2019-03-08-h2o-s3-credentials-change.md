---
title: "Connecting to multiple S3 buckets with H2O"
published: true
author: pavel_pscheidl
categories:
  - H2O-3
tags:
  - H2O.ai
  - H2O-3
  - Machine learning
  - AI
---

A common use-case when working with H<sub>2</sub>O open-source machine learning platform is to load data from Amazon S3 buckets. However, not all buckets are publicly accessible. In the case of H<sub>2</sub>O users loading data from a single Amazon S3 bucket, the traditional ways of providing secret credentials making the desired bucket accessible desribed in [H2O's documentation](https://docs.h2o.ai/h2o/latest-stable/h2o-docs/cloud-integration/ec2-and-s3.html) are sufficient. In general, H2O is able to pick up S3 credentials with a chain of providers, searching in the following locations:

- Credentials stored in `AwsCredentials.properties` file,
- From the EC2 instance itself, if H2O is running on it and the bucket is accessible by the same user,
- Environment variables,
- System properties `aws.accessKeyId` and `aws.secretKey`,
- Profile credentials provider.

For details, see the [documentation](https://docs.h2o.ai/h2o/latest-stable/h2o-docs/cloud-integration/ec2-and-s3.html).

However, there might be a problem when connecting to multiple S3 buckets during one session - all the above-mentioned options do load the S3 credentials during startup or the user has limited means of forcing a credential refresh.


## The solution

In order to make accessing multiple buckets with distinct credentials possible, one more credential provider with top priority has been introduced. For our users, this means calling a simple function in H<sub>2</sub>O API before accessing a bucket in time `t` with credentials  different from a bucket accessed in time `t-1`.

### Python example

```python
# Iris dataset from imaginary S3 bucket is about to be downloaded. There are no credentials set anywhere, so the call to set them is made right before the call.
from h2o.persist import set_s3_credentials
set_s3_credentials("ACCESSKEYID", "SECRETACCESSKEY")
iris = h2o.import_file("s3://test.somewhere.com/iris.csv")
airlines = h2o.import_file("s3://test.somewhere.com/airlines.csv")

# New bucket somewhere else is being accessed, set the correct credentials
set_s3_credentials("DIFFERENT/ACCESSKEYID", "DIFFERENT/SECRETACCESSKEY")
iris = h2o.import_file("s3://differenttest.somewhereelse.com/different-iris.csv")
```

### R example


```R
# Iris dataset from imaginary S3 bucket is about to be downloaded. There are no credentials set anywhere, so the call to set them is made right before the call.
h2o.set_s3_credentials("ACCESSKEYID", "SECRETACCESSKEY")
iris <- h2o.importFile(path = "s3://test.somewhere.com/iris.csv")
airlines <- h2o.importFile(path = "s3://test.somewhere.com/airlines.csv")

# New bucket somewhere else is being accessed, set the correct credentials
h2o.set_s3_credentials("DIFFERENT/ACCESSKEYID", "DIFFERENT/SECRETACCESSKEY")
iris <- h2o.importFile(path = "s3://differenttest.somewhereelse.com/different-iris.csv")
```