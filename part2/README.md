# Part 2 - Repository Scanning and Advanced Analysis

In this section, we will cover several techniques for repository scanning and perform advanced security analysis using tools like **GitHub Actions**, **TruffleHog**, **Tartufo**, **Horusec**, **CodeQL**, **Dependabot**, **Endor Labs**, and **SBOM generation**. By the end, you'll have an in-depth understanding of how to integrate and automate these security checks in a CI/CD pipeline.

## Technologies Covered:
1. **GitHub Native Secrets Scanning**
2. **TruffleHog**
3. **Tartufo**
4. **Horusec**
5. **Dependabot**
6. **CodeQL**
7. **Endor Labs Reachability Analysis**
8. **SBOM with VEX Reachability Data**
9. **Running SCA Locally**

---

## Module 4: Secrets Scanning

We will first explore secrets scanning using GitHub’s native tool and third-party tools like **TruffleHog** and **Tartufo**.

### GitHub Native Secrets Scanning

GitHub’s built-in secret scanning feature can detect hardcoded secrets, such as API keys, passwords, and tokens, in your repository.

#### Steps:
1. Navigate to your repository’s **Settings**.
2. Under **Code security and analysis**, enable **Secret scanning**.
3. Detected secrets will be listed under the **Security > Secret scanning** tab.

![Part 2 - Secret in GitHub](./img/secretsconfig.png "Secret in GitHub")

### TruffleHog and Tartufo

**TruffleHog** and **Tartufo** are additional tools that scan for high-entropy strings and report any suspected secrets.

#### TruffleHog Setup:
1. Install TruffleHog:
   ```bash
   pip install truffleHog
   ```

2. Run TruffleHog on your repository:
   ```bash
   trufflehog git https://github.com/YOUR-REPO-URL
   ```

#### Tartufo Setup:
1. Install Tartufo:
   ```bash
   pip install tartufo
   ```

2. Create a GitHub Actions workflow to run Tartufo:
   ```yaml
   name: Tartufo Secret Scanning

   on: [push, pull_request]

   jobs:
     scan:
       runs-on: ubuntu-latest
       steps:
       - name: Checkout Code
         uses: actions/checkout@v4
       
       - name: Run Tartufo
         uses: godaddy/tartufo-action@v4.1.0
   ```

Running Tartufo Locally:
```bash
tartufo scan --exclude-paths .git --branch main
```

---

### Horusec

Horusec is a comprehensive security tool that runs multiple scanners and aggregates their findings. It can be integrated into your GitHub Actions pipeline or run locally.

#### Steps to Integrate Horusec in GitHub Actions:
1. Create the following GitHub Actions workflow:
   ```yaml
   name: Horusec Security Scan

   on: [push, pull_request]

   jobs:
     scan:
       runs-on: ubuntu-latest
       steps:
       - name: Checkout Code
         uses: actions/checkout@v4
       
       - name: Run Horusec Scan
         uses: fike/horusec-action@v0.2.2
   ```

Horusec generates findings like the following:
```console
Language: Leaks
Severity: CRITICAL
Line: 41
Column: 0
SecurityTool: HorusecEngine
Confidence: MEDIUM
File: /github/workspace/part2/README.md
Code: -----BEGIN PRIVATE KEY-----(.|\s)*?-----END PRIVATE KEY-----
RuleID: HS-LEAKS-12
Type: Vulnerability
ReferenceHash: 943157e96e40574c92c9313d40a13689dc4c5b176d54e156c8e7c5f1a2ddd8ee
Details: (1/1) * Possible vulnerability detected: Asymmetric Private Key
Found SSH and/or x.509 Certificates among the files of your project. For more information, check out the CWE-312 advisory: https://cwe.mitre.org/data/definitions/312.html
```

---

## Module 5: Handling Secrets in GitHub

GitHub provides a secure method for storing secrets like API keys and passwords in a repository. These secrets can be accessed securely in your workflows without being exposed in your code.

#### Steps to Add Secrets:
1. Navigate to **Settings > Secrets and variables > Actions**.
2. Click **New repository secret**.
3. Add your secret, for example, `API_KEY`.

Using Secrets in a Workflow:
```yaml
name: Example Using Secrets

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout Code
      uses: actions/checkout@v4
    
    - name: Use API Key
      run: echo "Using API Key: ${{ secrets.API_KEY }}"
```

---

## Module 6: Detecting Security Vulnerabilities

GitHub provides tools like **CodeQL** and **Dependabot** to detect vulnerabilities in your codebase and its dependencies.

### CodeQL for Static Analysis

**CodeQL** is a powerful tool that helps detect security vulnerabilities through static analysis.

#### Steps to Enable CodeQL:
1. Create a workflow in `.github/workflows/codeql-analysis.yml`:
   ```yaml
   name: CodeQL Analysis

   on: [push, pull_request]

   jobs:
     analyze:
       runs-on: ubuntu-latest
       steps:
       - name: Checkout Code
         uses: actions/checkout@v4

       - name: Initialize CodeQL
         uses: github/codeql-action/init@v2

       - name: Perform CodeQL Analysis
         uses: github/codeql-action/analyze@v2
   ```

![Part 2 - Actions Workflows](./img/codeqlexample.png "Workflows")

This workflow will automatically scan the repository and report findings.

---

## Module 7: Vulnerable Dependencies (Dependabot)

**Dependabot** monitors your repository’s dependencies for known vulnerabilities and automatically creates pull requests to update them.

#### Steps to Enable Dependabot:
1. Go to `Settings > Code security and analysis`.
2. Enable **Dependabot alerts**.
3. Dependabot will automatically check for vulnerable dependencies and raise pull requests for updates.

---

## Module 8: Static Analysis (CodeQL)

We’ve already discussed setting up CodeQL. In this section, we will highlight how it integrates into your CI/CD pipeline, ensuring that each pull request is checked for vulnerabilities before merging.

---

## Module 9: Branch Protection Rules

Branch protection rules help ensure that certain checks (such as CodeQL and Dependabot) must pass before allowing any merges into protected branches. This enforces secure development practices.

#### Steps to Set Branch Protection Rules:
1. Navigate to **Settings > Branches**.
2. Set branch protection rules for `main`, requiring status checks like CodeQL analysis to pass before merging.

---

## Module 10: SBOMs (Software Bill of Materials)

A Software Bill of Materials (SBOM) provides a detailed list of components and dependencies used in your project. GitHub supports SBOM generation, which helps you track and assess security risks in your dependencies.

#### Steps:
1. Generate an SBOM using GitHub's SPDX tool:
   ```bash
   spdx-tool generate
   ```

Here is an example of an SBOM generated in JSON format:
```json
{
  "SPDXID": "SPDXRef-DOCUMENT",
  "spdxVersion": "SPDX-2.3",
  "creationInfo": {
    "created": "2024-03-20T18:26:13Z",
    "creators": [
      "Tool: GitHub.com-Dependency-Graph"
    ]
  },
  "name": "com.github.tweag/dev-sec-ops-workshop",
  "dataLicense": "CC0-1.0",
  "documentDescribes": [
    "SPDXRef-com.github.tweag-dev-sec-ops-workshop"
  ],
  "relationships": [
    {
      "relationshipType": "DEPENDS_ON",
      "spdxElementId": "SPDXRef-com.github.tweag-dev-sec-ops-workshop",
      "relatedSpdxElement": "SPDXRef-actions-github-codeql-action-init-3.*.*"
    },
    {
      "relationshipType": "DEPENDS_ON",
      "spdxElementId": "SPDXRef-com.github.tweag-dev-sec-ops-workshop",
      "relatedSpdxElement": "SPDXRef-actions-trufflesecurity-trufflehog-main"
    }
  ]
}
```

![Part 2 - SBOM and Dependencies](./img/sbom.png "SBOM and Dependencies")

---

## Module 11: Endor Labs Reachability Analysis

**Endor Labs**' reachability analysis helps prioritize vulnerabilities by identifying which vulnerabilities in your code are actually exploitable.

![Part 2 - SBOM and Endor Labs](./img/endorlabs.png "SBOM and Endor Labs")

We can integrate a repository in Endor labs through a number of simple steps.

If you haven't created a new account and Tenant, start by visiting the Endor Labs website, and signing up: https://www.endorlabs.com. You can sue your GitHub ID to sign up.

Once this is setup you will have a default Tenant and Namespace. For this demo we can stick with these. When you created your account, you will have been prompted to create a new tenant. Make a note of the name, as we will use this shortly.

Next we need to setup an Access Control policy, so that GitHub and Endor Labs can communicate. To do this:

1. Navigate to the Access Control menu listed in the bottom left of the screen.
2. Create a new `AUTH POLICY`
3. The `ID Provider` should be the GitHub Action option
4. You need to update the rule so that it should be `user=<your GitHub Org name>` make sure you add in your Org or user here
5. Permissions should be `Code Scanner`
6. Under `Advanced`, set the namespace to yours (this will be your tenant name), and select the `Propagate this policy to all child namespaces option` if you wish

You can now go ahead and add the following Action to run scans against the JS code in this repository, and push the results up to Endor Labs. Make sure to replace the namespace in this file with yours.

```yaml

name: "Endor Labs: Example Scan of JavaScript"
on:
  pull_request:
    branches: [ "main" ]
jobs:
  create_project_javascript:
    permissions:
      id-token: write # This is required for requesting the JWT
      contents: read  # Required by actions/checkout@v4 to checkout a private repository
      actions: read
      repository-projects: read
      pull-requests: read
    runs-on: ubuntu-latest
    steps:
      - name: 'Checkout Repository'
        uses: actions/checkout@v4
      - name: 'Use Node.js'
        uses: actions/setup-node@v4
        with:
          node-version: '20.x'
      - run: npm ci
        working-directory: ./part2/VulnerableAppTwo
      - run: npm run build --if-present
        working-directory: ./part2/VulnerableAppTwo
      - name: 'Scan JS with Endor Labs'
        uses: endorlabs/github-action@main # This workflow uses the Endor Labs GitHub action to scan.
        with:
          namespace: '<add your tenant namespace here>'
          pr: false
          scan_secrets: true
          scan_dependencies: true
          log_verbose: true
          scan_summary_output_type: 'table'
          sarif_file: 'findings.sarif'
      - name: 'Upload findings to GitHub'
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: 'findings.sarif'

```

When you create a new pull request, this Action will trigger.

You can try it out by opening this README file, and editing the following message from False to True:

```
Testing Endor Labs = False

```

Commit the change to this README to a new branch, and create a pull request and merge it.

---

## Module 12: Generating SBOM with VEX Reachability Data

By adding VEX (Vulnerability Exploitability eXchange) data to your SBOM, you can determine if the vulnerabilities found in your dependencies are exploitable within your code.

#### Steps:
1. Export your SBOM in SPDX format.
2. Enrich it with VEX data using Endor Labs’ tools.

---

## Module 13: Running SCA Locally

Running Software Composition Analysis (SCA) tests locally is essential to check your dependencies for known vulnerabilities before committing them to the repository.

#### Example Command:
```bash
scantool run --local
```

This will scan your project’s dependencies and report any vulnerabilities.

---

## Wrap-Up

In this workshop, we have covered:
- Secrets scanning with GitHub, TruffleHog, and Tartufo.
- Handling secrets securely with GitHub Actions.
- Vulnerability detection with CodeQL and Dependabot.
- Generating SBOMs with VEX data.
- Running local SCA tests to verify dependencies before pushing code.
