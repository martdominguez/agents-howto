Apply these instructions when working on backend controllers, DTOs, OpenAPI contracts, or Bruno files.

If an endpoint is added or changed:
- update or create the matching Bruno request
- update Bruno environment values if required
- keep request method, URL, headers, query params, path params, and payload in sync
- if the endpoint contract changed, review whether existing Bruno examples are now stale
- include Bruno files in the impacted file list before finishing
