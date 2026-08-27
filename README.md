# Inflectra/rapiselauncher-win-action

A GitHub Action to install the native Windows Rapise and run Rapise test sets stored in SpiraTest using `RapiseLauncher.exe`.

Requires a `windows-latest` (or self-hosted Windows) runner.

## Prerequisites

- A Spira instance (SpiraTest / SpiraTeam / SpiraPlan) with an active user account.
- Test Sets in Spira configured to run automated Rapise tests.
- An Automation Host record created in Spira (e.g., named `GHA`).

## Storing Spira Credentials

Store your Spira API Key as a GitHub Secret so it is never exposed in workflow files:

1. In Spira, go to **My Profile** and copy your **RSS Token** (this is your API Key).
2. In your GitHub repository, go to **Settings > Secrets and variables > Actions**.
3. Click **New repository secret**, set the name to `SPIRA_API_KEY`, paste your token, and save.

## Quick Start

```yaml
name: Run Rapise Tests

on:
  workflow_dispatch:

jobs:
  test:
    runs-on: windows-latest
    steps:
      - uses: Inflectra/rapiselauncher-win-action@v2
        with:
          spira_url: 'https://myserver.spiraservice.net/'
          spira_test_set_id: '925,1266'
          spira_username: 'myuser'
          spira_api_key: ${{ secrets.SPIRA_API_KEY }}
          spira_automation_host: GHA
```

## Usage

### Running with RepositoryConnection.xml

If you already have a `RepositoryConnection.xml` file checked into your repository (with Spira server, user, and password pre-configured), point the action to it:

```yaml
name: Run Rapise Tests (Config File)

on:
  workflow_dispatch:

jobs:
  test:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4

      - uses: Inflectra/rapiselauncher-win-action@v2
        with:
          spira_config: '${{ github.workspace }}/RepositoryConnection.xml'
          spira_test_set_id: '925,1266'
```

When `spira_config` is provided, the `spira_url`, `spira_username`, `spira_api_key`, and `spira_automation_host` inputs are not needed — everything is read from the XML file.

### Running with Test Set URL and Credentials

Pass Spira connection details directly. The `spira_url` input supports two forms:

- **Short form**: `https://myserver.spiraservice.net/` — requires `spira_project_id` and `spira_test_set_id` to be set separately.
- **Full form**: `https://myserver.spiraservice.net/9/TestSet/925.aspx` — the project ID and test set ID are extracted automatically from the URL.

```yaml
name: Run Rapise Tests (Credentials)

on:
  workflow_dispatch:

jobs:
  test:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4

      - uses: Inflectra/rapiselauncher-win-action@v2
        with:
          spira_url: 'https://myserver.spiraservice.net/9/TestSet/925.aspx'
          spira_username: 'myuser'
          spira_api_key: ${{ secrets.SPIRA_API_KEY }}
          spira_automation_host: GHA
```

### Enabling Video Recording

Record a video of the test execution and upload it to the Spira Test Run:

```yaml
      - uses: Inflectra/rapiselauncher-win-action@v2
        with:
          spira_url: 'https://myserver.spiraservice.net/9/TestSet/925.aspx'
          spira_username: 'myuser'
          spira_api_key: ${{ secrets.SPIRA_API_KEY }}
          record_video: 'true'
          record_video_options: '-noaudio -bitRate 512 -frameRate 2'
```

The default `record_video_options` are `-noaudio -bitRate 512 -frameRate 2`. Adjust as needed.

### Setting Screen Resolution

Set a custom screen resolution for the runner before test execution:

```yaml
      - uses: Inflectra/rapiselauncher-win-action@v2
        with:
          spira_url: 'https://myserver.spiraservice.net/9/TestSet/925.aspx'
          spira_username: 'myuser'
          spira_api_key: ${{ secrets.SPIRA_API_KEY }}
          set_screen_size: 'true'
          screen_width: '1920'
          screen_height: '1080'
```

### Video Recording with Custom Screen Size

Combine both features for full desktop UI test coverage:

```yaml
      - uses: Inflectra/rapiselauncher-win-action@v2
        with:
          spira_url: 'https://myserver.spiraservice.net/9/TestSet/925.aspx'
          spira_username: 'myuser'
          spira_api_key: ${{ secrets.SPIRA_API_KEY }}
          set_screen_size: 'true'
          screen_width: '1920'
          screen_height: '1080'
          record_video: 'true'
```

### Screenshots

By default (`capture_screenshots: 'true'`), the action captures a screenshot of the desktop before and after test execution. These are included in the uploaded artifacts. Disable with:

```yaml
capture_screenshots: 'false'
```

### Setting a Timeout

Use `timeout_minutes` to kill the launcher if it exceeds the specified duration:

```yaml
      - uses: Inflectra/rapiselauncher-win-action@v2
        with:
          spira_url: 'https://myserver.spiraservice.net/9/TestSet/925.aspx'
          spira_username: 'myuser'
          spira_api_key: ${{ secrets.SPIRA_API_KEY }}
          timeout_minutes: 10
```

### Specifying Rapise Version

```yaml
      - uses: Inflectra/rapiselauncher-win-action@v2
        with:
          rapise_version: '9.0.35.37'
          # ... other inputs
```

### Git Root for Spira Tests Stored in Git

If your Rapise tests are stored in a Git repository connected to Spira, the action automatically sets `GITROOT` to `$GITHUB_WORKSPACE`. Override it with `git_root` if needed:

```yaml
      - uses: Inflectra/rapiselauncher-win-action@v2
        with:
          git_root: '${{ github.workspace }}/tests'
          # ... other inputs
```

### Artifacts

By default (`upload_artifacts: 'true'`), the action collects screenshots, `.trp`, `.tap`, `.log`, `.wmv` files, and Rapise logs into a `rapise-results` artifact. Disable with:

```yaml
upload_artifacts: 'false'
```

## Input Reference

| Input | Default | Description |
|---|---|---|
| `spira_config` | | Path to existing `RepositoryConnection.xml`. When set, URL/user/key inputs are ignored. |
| `spira_url` | | Spira server URL. Short form: `https://server/`. Full form: `https://server/9/TestSet/925.aspx`. |
| `spira_username` | | Spira username. |
| `spira_api_key` | | Spira API key (RSS Token). |
| `spira_project_id` | | Spira Project ID. Optional if using full-form URL. |
| `spira_test_set_id` | | Test Set ID(s), comma-separated (e.g. `925,1266`). |
| `spira_automation_host` | *(hostname)* | Automation Host Token. Defaults to runner hostname. |
| `install_rapise` | `true` | Whether to install Rapise. |
| `rapise_version` | `9.0.35.37` | Rapise version to install. |
| `set_screen_size` | `false` | Set screen resolution before execution. |
| `screen_width` | `1920` | Screen width (1024–7680). |
| `screen_height` | `1080` | Screen height (768–4320). |
| `record_video` | `false` | Record video and upload to Spira Test Run. |
| `record_video_options` | `-noaudio -bitRate 512 -frameRate 2` | Video recording options. |
| `capture_screenshots` | `true` | Capture before/after desktop screenshots. |
| `upload_artifacts` | `true` | Upload execution artifacts after run. |
| `timeout_minutes` | `0` | Execution timeout in minutes. `0` = no timeout. |
| `git_root` | | Path to Git project root. Defaults to `$GITHUB_WORKSPACE`. |

## Reviewing Results

- **In Spira**: Test execution results are automatically uploaded to your Test Run records.
- **In GitHub**: After the workflow completes, go to the workflow run summary page. Under the **Artifacts** section, download the `rapise-results` archive containing logs, reports, screenshots, and video files for troubleshooting.

## See Also

- [Inflectra/rapiselauncher-node-action](https://github.com/Inflectra/rapiselauncher-node-action) — cross-platform (Linux & Windows) action using the Node.js Rapise engine.
