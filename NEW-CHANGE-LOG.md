All changes are complete. Here's a summary of what was modified:
---
Changes in library_configuration.py
- folio_username and folio_password are now optional (default "") since they're not needed for keycloak auth.
- Added 4 new fields to LibraryConfiguration:
  - auth_type (default "legacy") -- "legacy" for username/password, "keycloak" for client credentials
  - client_id (default "") -- required when auth_type is "keycloak"
  - client_secret (default "") -- required when auth_type is "keycloak"
  - auth_base_url (default "") -- keycloak server URL, falls back to gateway_url
- Added validate_auth_credentials validator that ensures the correct credential set is provided based on auth_type.
Changes in __main__.py
- New CLI arguments (all with env var fallbacks, none prompted interactively):
  - --auth_type / FOLIO_MIGRATION_TOOLS_AUTH_TYPE (default "legacy")
  - --client_id / FOLIO_MIGRATION_TOOLS_CLIENT_ID
  - --client_secret / FOLIO_MIGRATION_TOOLS_CLIENT_SECRET (secure input)
  - --auth_base_url / FOLIO_MIGRATION_TOOLS_AUTH_BASE_URL
- prep_library_config() injects keycloak args into config (same pattern as existing password injection -- CLI args are only used if not already in the config file).
- FolioClient instantiation branches on auth_type:
  - "keycloak": passes client_id, client_secret, auth_type="keycloak", auth_base_url, api_base_url
  - "legacy": passes folio_username, folio_password (unchanged behavior)
- Removed the 4 print() statements that were dumping gateway_url, tenant_id, folio_username, and folio_password to stdout.
How users can configure keycloak auth
Option 1 -- Config file (JSON, camelCase):
{
  libraryInformation: {
    gatewayUrl: https://folio.example.com,
    tenantId: my_tenant,
    authType: keycloak,
    clientId: my-client-id,
    clientSecret: my-secret,
    authBaseUrl: https://keycloak.example.com,
    ...
  }
}
Option 2 -- CLI args:
folio-migration-tools config.json MyTask \
  --auth_type keycloak \
  --client_id my-client-id \
  --client_secret my-secret \
  --auth_base_url https://keycloak.example.com

  
Option 3 -- Environment variables:
export FOLIO_MIGRATION_TOOLS_AUTH_TYPE=keycloak
export FOLIO_MIGRATION_TOOLS_CLIENT_ID=my-client-id
export FOLIO_MIGRATION_TOOLS_CLIENT_SECRET=my-secret
export FOLIO_MIGRATION_TOOLS_AUTH_BASE_URL=https://keycloak.example.com
All three can be mixed -- config file values take precedence over CLI args/env vars (consistent with the existing password handling pattern).