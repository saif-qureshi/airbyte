# Excel Sheets Source

This is the repository for the Excel Sheets source connector, written in declarative YAML using the Airbyte CDK.
For information about how to use this connector within Airbyte, see [the documentation](https://docs.airbyte.com/integrations/sources/excel-sheets).

## Local development

### Prerequisites

- Python 3.9+
- Poetry
- An Excel workbook stored in OneDrive or SharePoint
- Microsoft Azure app registration with appropriate permissions

### Installing the connector

From this connector directory, run:
```bash
poetry install --with dev
```

### Authentication Setup

This connector uses Microsoft Graph API to access Excel workbooks and supports two authentication methods:

#### 1. OAuth2.0 (Delegated Permissions) - For Personal Accounts
Best for accessing files that a specific user has access to.

1. Register an application in Azure AD
2. Grant the following **delegated** permissions:
   - Files.Read or Files.Read.All
   - Sites.Read.All (for SharePoint)
   - offline_access (for refresh token)
3. Configure OAuth2.0 authentication with user consent flow
4. Obtain a refresh token through the OAuth flow

#### 2. Service Principal (Application Permissions) - For Service Accounts
Best for automated/scheduled syncs without user interaction.

1. Register an application in Azure AD
2. Grant the following **application** permissions:
   - Files.Read.All
   - Sites.Read.All
3. Admin consent is required for these permissions
4. Use client credentials (no refresh token needed)

### Testing

#### Run unit tests

From the connector directory, run:
```bash
poetry run pytest unit_tests
```

#### Run integration tests

From the connector directory, run:
```bash
poetry run pytest integration_tests
```

### Using the connector

Create a configuration file with your Excel workbook details:

#### For OAuth2.0 (Delegated Permissions):
```json
{
  "workbook_id": "https://contoso.sharepoint.com/sites/TeamSite/Shared%20Documents/MyWorkbook.xlsx",
  "credentials": {
    "auth_type": "OAuth",
    "tenant_id": "common",
    "client_id": "your-client-id",
    "client_secret": "your-client-secret",
    "refresh_token": "your-refresh-token"
  },
  "batch_size": 1000000,
  "names_conversion": false
}
```

#### For Service Principal (Application Permissions):
```json
{
  "workbook_id": "https://contoso.sharepoint.com/sites/TeamSite/Shared%20Documents/MyWorkbook.xlsx",
  "credentials": {
    "auth_type": "Service",
    "tenant_id": "your-tenant-id",
    "client_id": "your-client-id",
    "client_secret": "your-client-secret"
  },
  "batch_size": 1000000,
  "names_conversion": false
}
```

Then run the connector:
```bash
poetry run source-excel-sheets spec
poetry run source-excel-sheets check --config secrets/config.json
poetry run source-excel-sheets discover --config secrets/config.json
poetry run source-excel-sheets read --config secrets/config.json --catalog sample_files/configured_catalog.json
```

## Features

- Reads Excel workbooks from OneDrive and SharePoint
- Each worksheet becomes a separate stream
- Supports batch processing for large worksheets
- Column name sanitization options
- Stream name overrides
- OAuth2.0 authentication

## Limitations

- Only supports Excel files stored in OneDrive or SharePoint (not local files)
- All data is treated as strings (no type inference)
- Requires first row to contain headers
- No incremental sync support