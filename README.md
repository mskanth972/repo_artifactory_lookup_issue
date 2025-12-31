# Renovate Bug Reproduction: github-releases uses python registryUrls

This repository demonstrates a bug where Renovate's `github-releases` datasource incorrectly uses Python `registryUrls` configuration when looking up `containerbase/python-prebuild`.

## Bug Description
When Renovate tries to update Python versions via `github-releases` datasource for `containerbase/python-prebuild`, it incorrectly uses the configured Python `registryUrls` instead of GitHub's API endpoint.

## Steps to Reproduce

### 1. Clone and setup Renovate
```bash
git clone https://github.com/renovatebot/renovate.git
cd renovate
git checkout 41.143.1

# Install Node.js v22
fnm install v22
fnm use v22

# Install dependencies
pnpm install
```

### 2. Create config.js
```
// config.js
'use strict';

module.exports = {
  // Python Support
  "python": {
    "registryUrls": ["https://pypi.org/simple/"],
  },

  // Host Rules for GitHub.com
  "hostRules": [
    {
      "matchHost": "github.com",
      "token": process.env.GITHUB_COM_TOKEN
    }
  ],

  // Optional performance settings
  "persistRepoData": true,
  "repositoryCache": "enabled"
};
```
### 3. Set environment variables
Create a .env file:

```bash
GITHUB_COM_TOKEN="your_github_com_personal_access_token"
LOG_LEVEL=debug
NODE_OPTIONS="--max-old-space-size=7168"
```
Load the environment:

```bash
export $(xargs < .env)
```
### 4. Optional: Backup git config (if experiencing auth issues)
```bash
mv ~/.gitconfig ~/.gitbackup 2>/dev/null || true
```
### 5. Run Renovate against this repository
```bash
pnpm start mskanth972/repo_artifactory_lookup_issue
```

## Expected Behavior
Renovate should call GitHub API (https://api.github.com/graphql) when looking up containerbase/python-prebuild via github-releases datasource.

## Actual Behavior
Renovate calls Python registry URL + `/api/graphql`:

```log
POST https://pypi.org/simple/api/graphql 404 306 1
DEBUG: Datasource connection error
       "datasource": "github-releases",
       "packageName": "containerbase/python-prebuild",
       "url": "https://pypi.org/simple/api/graphql",
       "errCode": "ERR_NON_2XX_3XX_RESPONSE"
```
## Evidence in This Repository
PR [#1](https://github.com/mskanth972/repo_artifactory_lookup_issue/pull/1) shows: Failed to look up github-releases package containerbase/python-prebuild

Files that trigger the bug:

- .python-version - Contains Python version 3.13

- pyproject.toml - Poetry configuration with Python dependency

### Same Issue with Internal Registries
The same bug occurs with internal/custom Python registries (e.g., Artifactory, Nexus, etc.):

`
https://internal-artifactory.example.com/artifactory/api/pypi/python/simple/api/graphql
`

