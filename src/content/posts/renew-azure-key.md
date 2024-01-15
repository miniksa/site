---
title: "Renewing AzureKeyVault key for site"
date: 2024-01-14T19:10:00-08:00
draft: false
tags: ["site"]
---

Lo and behold the key in the KeyVault expired for the site.

Here's how I fixed it.

## The Error
```
##[error]Could not fetch access token for Azure. Verify if the Service Principal used is valid and not expired.
##[error]Get secrets failed. Error: Could not fetch access token for Azure. Status code: invalid_client, status message: 7000222 - [2024-01-15 03:08:33Z]: AADSTS7000222: The provided client secret keys for app '***' are expired. Visit the Azure portal to create new keys for your app: https://aka.ms/NewClientSecret, or consider using certificate credentials for added security: https://aka.ms/certCreds. 
```

## The Solution
1. Realize that it's Azure DevOps trying to get TO the Key Vault that has expired and not the connection string itself inside the vault to the blob.
1. Find [this StackOverflow](https://stackoverflow.com/questions/70777248/getting-error-could-not-fetch-access-token-for-azure-when-deploying-using-azure)
1. Go to `https://<organization>.visualstudio.com/<project>/_settings/adminservices` in the URL bar. I couldn't find a button for it, I just slammed `_settings` at the end of the URL in various positions until it got me here.
1. Click on the Azure connection so it opens full page
1. Click `Edit` in the top right corner.
1. Put some garbage in the `Description (optional)` field to make it change and click `Save`.
1. A log-in window pops up on top. Log in with the *Microsoft Account* attached to the Azure Subscription.
1. When that finishes, write this line then push it to the site to see if it actually deploys correctly.