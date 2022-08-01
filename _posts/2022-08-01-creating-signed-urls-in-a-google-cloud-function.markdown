---
layout: post
title:  "Creating signed urls in a google cloud function"
date:   2022-08-01 15:47:10 +0200
categories: google-cloud-function external-memory 
---

[Signed urls](https://cloud.google.com/storage/docs/access-control/signed-urls) are a neat way of sharing private data for a limited time. However, with google cloud functions it can be 
[difficult](https://blog.bavard.ai/how-to-generate-signed-urls-using-python-in-google-cloud-run-835ddad5366)
[to](https://github.com/googleapis/google-auth-library-python/issues/338) 
[get](https://stackoverflow.com/q/65568916)
[to](https://bytemeta.vip/repo/google-github-actions/auth/issues/164)
[work](https://stackoverflow.com/q/64234214).
All these posts have the same error. 

```
AttributeError: you need a private key to sign credentials.the credentials you are currently using <class 'google.auth.impersonated_credentials.Credentials'> just contains a token. see https://google-cloud-python.readthedocs.io/en/latest/core/auth.html?highlight=authentication#setting-up-a-service-account for more details.
```

The [documentation](https://cloud.google.com/storage/docs/access-control/signing-urls-with-helpers#code-samples) for creating signed urls on with cloud functions is non-existing. 

In the end, I got it to work due to [this excellent post](https://blog.bavard.ai/how-to-generate-signed-urls-using-python-in-google-cloud-run-835ddad5366) by Evan Peterson. 

```python
from datetime import timedelta

from google import auth
from google.auth.transport import requests
from google.cloud.storage import Client


credentials, project_id = auth.default()
if credentials.token is None:
    # Perform a refresh request to populate the access token of the
    # current credentials.
    credentials.refresh(requests.Request())

client = Client()
bucket = client.get_bucket(bucket)
blob = bucket.blob(blob)
return blob.generate_signed_url(
    version="v4",
    expiration=timedelta(hours=1),
    service_account_email=credentials.service_account_email,
    access_token=credentials.token,
    method="GET",
)
```

Also make sure that your service account that runs the cloud function has the `roles/iam.serviceAccountTokenCreator` role. 
