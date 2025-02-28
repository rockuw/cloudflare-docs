---
pcx-content-type: reference
title: Common API calls
weight: 10
---

# Common API calls

As a SaaS provider, you may want to configure and manage Cloudflare for SaaS [via the API](https://api.cloudflare.com/) rather than the [Cloudflare dashboard](https://dash.cloudflare.com/). Below are relevant API calls for creating, editing, and deleting custom hostnames, as well as monitoring, updating, and deleting fallback origins. Further details can be found in the [Cloudflare API documentation](https://api.cloudflare.com/).

---

## Custom hostnames

| Endpoint                                                                                                                                 | Notes                                                                                                                                   |
| ---------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| [List custom hostnames](https://api.cloudflare.com/#custom-hostname-for-a-zone-list-custom-hostnames)                                    | Use the `page` parameter to pull additional pages. Add a `hostname` parameter to search for specific hostnames.                         |
| [Create custom hostname](https://api.cloudflare.com/#custom-hostname-for-a-zone-create-custom-hostname)                                  | In the `validation_records` object of the response, use the `txt_name` and `txt_record` listed  to validate the custom hostname. |
| [Custom hostname details](https://api.cloudflare.com/#custom-hostname-for-a-zone-custom-hostname-details)                                |
| [Edit custom hostname](https://api.cloudflare.com/#custom-hostname-for-a-zone-edit-custom-hostname)                                      | When sent with an `ssl` object that matches the existing value, indicates that hostname should restart domain control validation (DCV). |
| [Delete custom hostname](https://api.cloudflare.com/#custom-hostname-for-a-zone-delete-custom-hostname-and-any-issued-ssl-certificates-) | Also deletes any associated SSL/TLS certificates.                                                                                       |

## Fallback origins

Our API includes the following endpoints related to the [fallback origin](/cloudflare-for-saas/getting-started/#step-1--create-fallback-origin-and-cname-target) of a custom hostname:

- [Get fallback origin](https://api.cloudflare.com/#custom-hostname-fallback-origin-for-a-zone-get-fallback-origin-for-custom-hostnames)
- [Update fallback origin](https://api.cloudflare.com/#custom-hostname-fallback-origin-for-a-zone-update-fallback-origin-for-custom-hostnames)
- [Remove fallback origin](https://api.cloudflare.com/#custom-hostname-fallback-origin-for-a-zone-delete-fallback-origin-for-custom-hostnames)
