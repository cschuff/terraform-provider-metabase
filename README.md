# Terraform Metabase provider

This repository contains the source code for the [Metabase](https://www.metabase.com/) Terraform provider.

For how to use the provider in a Terraform project, please refer to the [Terraform registry documentation](https://registry.terraform.io/providers/flovouin/metabase/latest/docs).

## Metabase version compatibility

Unfortunately, this provider relies on the Metabase API which is [subject to breaking changes and not versioned](https://www.metabase.com/docs/latest/api-documentation#about-the-metabase-api). This makes it hard for this provider to keep up with Metabase versions, apologies for that. Here is a table that summarizes supported Metabase versions:

| Provider version \ Metabase version | .44 | .45 | .46 | .47 | .48 | .49 | .50 |
| ----------------------------------: | :-: | :-: | :-: | :-: | :-: | :-: | :-: |
|                              <= 0.3 | ✅  | ❌  | ❌  | ❌  | ❌  | ❌  | ❌  |
|                       >= 0.4, < 0.8 | ❌  | ❌  | ❌  | ❌  | ✅  | ✅  | ❌  |
|                              >= 0.8 | ❌  | ❌  | ❌  | ❌  | ❌  | ❌  | ✅  |

## 🔨 `mbtf` importer tool

The `mbtf` command line allows importing Metabase dashboards and cards from a Metabase instance (API).

### Installation

#### Option 1: Download from GitHub Releases (Recommended)

Pre-built binaries are available for each release on the [GitHub Releases page](https://github.com/flovouin/terraform-provider-metabase/releases).

1. Download the appropriate archive for your operating system and architecture (e.g., `mbtf_1.0.0_linux_amd64.zip` for Linux on x86_64)
2. Extract the archive
3. Move the `mbtf` binary (or `mbtf.exe` on Windows) to a directory in your `PATH`, or use it directly

**Example for Linux/macOS:**
```bash
# Download and extract (replace VERSION and OS/ARCH with your values)
wget https://github.com/flovouin/terraform-provider-metabase/releases/download/v1.0.0/mbtf_1.0.0_linux_amd64.zip
unzip mbtf_1.0.0_linux_amd64.zip

# Make it executable and move to a directory in PATH
chmod +x mbtf
sudo mv mbtf /usr/local/bin/
```

**Example for Windows (PowerShell):**
```powershell
# Download and extract (replace VERSION with your values)
Invoke-WebRequest -Uri "https://github.com/flovouin/terraform-provider-metabase/releases/download/v1.0.0/mbtf_1.0.0_windows_amd64.zip" -OutFile "mbtf.zip"
Expand-Archive -Path "mbtf.zip" -DestinationPath "."
# Add the directory containing mbtf.exe to your PATH, or use it directly
```

#### Option 2: Run Directly from Release (One-liner)

You can download and run `mbtf` directly without installing it permanently:

**Linux/macOS:**
```bash
# Replace VERSION, OS, and ARCH with your values (e.g., v1.0.0, linux, amd64)
VERSION="v1.0.0" OS="linux" ARCH="amd64" && \
curl -L "https://github.com/flovouin/terraform-provider-metabase/releases/download/${VERSION}/mbtf_${VERSION#v}_${OS}_${ARCH}.zip" -o /tmp/mbtf.zip && \
unzip -o /tmp/mbtf.zip -d /tmp && \
chmod +x /tmp/mbtf && \
/tmp/mbtf "$@" && \
rm /tmp/mbtf /tmp/mbtf.zip
```

Or as a simpler one-liner (downloads to current directory):
```bash
# Linux x86_64
curl -L https://github.com/flovouin/terraform-provider-metabase/releases/download/v1.0.0/mbtf_1.0.0_linux_amd64.zip -o mbtf.zip && unzip -o mbtf.zip && chmod +x mbtf && ./mbtf && rm mbtf mbtf.zip

# macOS (Intel)
curl -L https://github.com/flovouin/terraform-provider-metabase/releases/download/v1.0.0/mbtf_1.0.0_darwin_amd64.zip -o mbtf.zip && unzip -o mbtf.zip && chmod +x mbtf && ./mbtf && rm mbtf mbtf.zip

# macOS (Apple Silicon)
curl -L https://github.com/flovouin/terraform-provider-metabase/releases/download/v1.0.0/mbtf_1.0.0_darwin_arm64.zip -o mbtf.zip && unzip -o mbtf.zip && chmod +x mbtf && ./mbtf && rm mbtf mbtf.zip
```

**Windows (PowerShell):**
```powershell
# Replace VERSION with your desired version (e.g., v1.0.0)
$VERSION = "v1.0.0"
$OS = "windows"
$ARCH = "amd64"
Invoke-WebRequest -Uri "https://github.com/flovouin/terraform-provider-metabase/releases/download/$VERSION/mbtf_$($VERSION.TrimStart('v'))_${OS}_${ARCH}.zip" -OutFile "mbtf.zip"
Expand-Archive -Path "mbtf.zip" -DestinationPath "." -Force
.\mbtf.exe
Remove-Item "mbtf.zip", "mbtf.exe"
```

**Note:** Replace `v1.0.0` with the actual release version you want to use. You can find the latest version on the [GitHub Releases page](https://github.com/flovouin/terraform-provider-metabase/releases).

#### Option 3: Build from Source

Alternatively, you can install the tool using the `go` command:

```bash
export PATH=${PATH}:$(go env GOPATH)/bin
cd cmd/mbtf
go install
```

### Running

Simply run the `mbtf` utility with no argument. A configuration should be defined first (see next section).

```bash
mbtf
```

Running the tool will connect to the Metabase API, list all dashboards matching the filter defined in the configuration, and import the dashboards and cards as Terraform files.

### Configuration

`mbtf` is configured using a single YAML file that should be located in the current directory where `mbtf` is run, with the name `mbtf.yml`. This file has the following structure:

```yaml
# Configures how to connect to Metabase.
metabase:
  # The URL to the Metabase API. Usually suffixed with `/api`.
  endpoint: http://metabase-endpoint.com/api
  # The following authentication parameters can be defined as environment variables:
  # - To use username/password authentication, set `MBTF_METABASE_USERNAME` and `MBTF_METABASE_PASSWORD`.
  # - To use API key authentication, set `MBTF_METABASE_APIKEY`.
  # Only one authentication method is required.
  username: email@address.com
  password: password
  # apikey: your_metabase_api_key

# Databases are not imported by `mbtf` and should already be defined in the Terraform configuration.
# This defines how the mapping is made between databases found in the Metabase API and Terraform.
databases:
  mapping:
    # A database name `Big Query` in the Metabase API will be referenced as `metabase_database.bigquery` in the
    # imported cards and dashboards.
    - name: Big Query
      resource_name: bigquery
    # A database can also be referenced by its ID in the Metabase API.
    - id: 23
      resource_name: postgres

# Similarly to databases, collections are not imported by `mbtf` and a mapping between the Metabase API and Terraform
# should be provided.
collections:
  mapping:
    # The collection with ID `193` in the Metabase API will be referenced as `metabase_collection.my_collection` in the
    # imported cards and dashboards. The `id` can also be `root` for the default collection.
    - id: 193
      resource_name: my_collection
    # A collection can also be referenced by its name in Metabase.
    - name: Other collection
      resource_name: other_collection

# Determines which dashboards should be imported.
dashboard_filter:
  # The list of collections for which dashboards should be imported.
  # All collections are imported by default if this is not specified.
  included_collections:
    - id: 53 # Just like for collections mapping, this can be `root` to include the default collection.
  # The list of collections to exclude from the import. This takes precedence over `included_collections`. However,
  # there is no point to define both included and excluded collections.
  excluded_collections:
    # Collections can also be filtered by name.
    - name: Private collection

  # A regexp that the dashboard name should match in order to be imported.
  dashboard_name: ^\[Public\]
  # A regexp that the dashboard description should match in order to be imported.
  dashboard_description: tag:reviewed

  # The list of IDs of the dashboards to import. If this is non-empty, all other parameters are ignored.
  dashboard_ids: [2, 3, 4]

# Defines how the Terraform configuration is written to files.
output:
  # The path where the Terraform configuration will be written.
  path: ./metabase
  # Whether generated files matching `mb-gen-*.tf` should be removed from the output directory before writing.
  clear: true
  # When `true`, `terraform fmt` is not called after writing the Terraform files.
  disable_formatting: false
```

## 🧑‍💻 Development

### Provider source

The source code provider is mainly located in the [internal/provider](./internal/provider/) folder. Each resource and data source has its own source file.

### Metabase API client

The source for the Metabase API client is located in the [metabase](./metabase/) folder and is mainly generated automatically from the [OpenAPI definition](./metabase-api.yaml). When updating the definition, the client can be regenerated using the `go generate` command.

### `mbtf` importer

The source for the Metabase to Terraform importer is located in the [internal/importer](./internal/importer/) folder, and its entrypoint is in [cmd/mbtf](./cmd/mbtf/). The `mbtf` importer uses the Metabase API client to retrieve resources and convert them to Terraform definitions.
