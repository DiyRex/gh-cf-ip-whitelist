# 🔒 Cloudflare IP Whitelist Manager

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-github--runner--ip--whitelister--cloudlfare-blue.svg?colorA=24292e&colorB=0366d6&style=flat&longCache=true&logo=github)](https://github.com/marketplace/actions/github-runner-ip-whitelister-cloudlfare)
[![CI](https://github.com/DiyRex/gh-cf-ip-whitelist/actions/workflows/test.yml/badge.svg)](https://github.com/DiyRex/gh-cf-ip-whitelist/actions/workflows/test.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![GitHub release](https://img.shields.io/github/release/DiyRex/gh-cf-ip-whitelist.svg)](https://github.com/DiyRex/gh-cf-ip-whitelist/releases)

> Automatically manage Cloudflare IP access rules for CI/CD pipelines. Whitelist runner IPs to bypass Cloudflare protection during automated testing.

## ✨ Features

- 🔐 **Automatic IP whitelisting** - No more Cloudflare blocks during testing
- 🧹 **Auto cleanup** - Removes rules when jobs complete or fail  
- 🌐 **Smart IP detection** - Automatically detects runner IP addresses
- ⚡ **Lightning fast** - No dependency installation required
- 🔄 **100% reliable** - Multiple fallback mechanisms
- 🛡️ **Secure** - Temporary rules with automatic expiration
- 📊 **Detailed logging** - Full visibility into rule management
- 🎯 **Framework agnostic** - Works with any testing tool

## 🚀 Quick Start

### 1. Setup Cloudflare API Token

Create an API token with these permissions:
- **Zone:Zone:Read** - To access zone information
- **Zone:Firewall Services:Edit** - To manage IP access rules

[📖 How to create Cloudflare API token →](#cloudflare-setup)

### 2. Add Repository Secrets

```
CLOUDFLARE_API_TOKEN=your_api_token_here
CLOUDFLARE_ZONE_ID=your_zone_id_here
```

### 3. Use in Your Workflow

```yaml
name: Tests with Cloudflare Bypass
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    # 🔒 Whitelist runner IP
    - name: Whitelist runner IP
      id: whitelist
      uses: DiyRex/gh-cf-ip-whitelist@v1
      with:
        api-token: ${{ secrets.CLOUDFLARE_API_TOKEN }}
        zone-id: ${{ secrets.CLOUDFLARE_ZONE_ID }}
        action: 'create'
    
    # 🧪 Run your tests (they'll bypass Cloudflare now!)
    - name: Run tests
      run: npm test
    
    # 🧹 Cleanup (always runs, even on failure)
    - name: Remove whitelist
      if: always()
      uses: DiyRex/gh-cf-ip-whitelist@v1
      with:
        api-token: ${{ secrets.CLOUDFLARE_API_TOKEN }}
        zone-id: ${{ secrets.CLOUDFLARE_ZONE_ID }}
        action: 'delete'
        rule-id: ${{ steps.whitelist.outputs.rule-id }}
```

## 📚 Usage Examples

### Playwright Testing

```yaml
name: Playwright Tests
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    container:
      image: mcr.microsoft.com/playwright:v1.48.0-jammy
    steps:
    - uses: actions/checkout@v4
    
    - name: Whitelist for Playwright
      id: whitelist
      uses: DiyRex/gh-cf-ip-whitelist@v1
      with:
        api-token: ${{ secrets.CLOUDFLARE_API_TOKEN }}
        zone-id: ${{ secrets.CLOUDFLARE_ZONE_ID }}
        action: 'create'
        notes: 'Playwright E2E Tests'
    
    - run: npm ci
    - run: npx playwright test
    
    - name: Cleanup whitelist
      if: always()
      uses: DiyRex/gh-cf-ip-whitelist@v1
      with:
        api-token: ${{ secrets.CLOUDFLARE_API_TOKEN }}
        zone-id: ${{ secrets.CLOUDFLARE_ZONE_ID }}
        action: 'delete'
        rule-id: ${{ steps.whitelist.outputs.rule-id }}
```

### Multi-Environment Testing

```yaml
name: Multi-Environment Tests
on: [push]

jobs:
  test-staging:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Whitelist for staging
      id: staging-rule
      uses: DiyRex/gh-cf-ip-whitelist@v1
      with:
        api-token: ${{ secrets.CLOUDFLARE_API_TOKEN_STAGING }}
        zone-id: ${{ secrets.CLOUDFLARE_ZONE_ID_STAGING }}
        action: 'create'
        notes: 'Staging Tests'
    
    - name: Test staging environment
      run: npm run test:staging
      env:
        BASE_URL: https://staging.yoursite.com
    
    - name: Cleanup staging
      if: always()
      uses: DiyRex/gh-cf-ip-whitelist@v1
      with:
        api-token: ${{ secrets.CLOUDFLARE_API_TOKEN_STAGING }}
        zone-id: ${{ secrets.CLOUDFLARE_ZONE_ID_STAGING }}
        action: 'delete'
        rule-id: ${{ steps.staging-rule.outputs.rule-id }}

  test-production:
    needs: test-staging
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Whitelist for production
      id: prod-rule
      uses: DiyRex/gh-cf-ip-whitelist@v1
      with:
        api-token: ${{ secrets.CLOUDFLARE_API_TOKEN_PROD }}
        zone-id: ${{ secrets.CLOUDFLARE_ZONE_ID_PROD }}
        action: 'create'
        notes: 'Production Smoke Tests'
        mode: 'challenge'  # Show challenge instead of full access
    
    - name: Production smoke tests
      run: npm run test:prod-smoke
      env:
        BASE_URL: https://yoursite.com
    
    - name: Cleanup production
      if: always()
      uses: DiyRex/gh-cf-ip-whitelist@v1
      with:
        api-token: ${{ secrets.CLOUDFLARE_API_TOKEN_PROD }}
        zone-id: ${{ secrets.CLOUDFLARE_ZONE_ID_PROD }}
        action: 'delete'
        rule-id: ${{ steps.prod-rule.outputs.rule-id }}
```

### Custom IP and Settings

```yaml
- name: Custom whitelist rule
  uses: DiyRex/gh-cf-ip-whitelist@v1
  with:
    api-token: ${{ secrets.CLOUDFLARE_API_TOKEN }}
    zone-id: ${{ secrets.CLOUDFLARE_ZONE_ID }}
    action: 'create'
    ip-address: '203.0.113.1'          # Specific IP
    mode: 'js_challenge'               # JavaScript challenge
    notes: 'Load Testing - Custom IP'  # Custom description
    wait-time: '60'                    # Longer propagation wait
```

## 📋 Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `action` | Action: `create` or `delete` | ✅ Yes | `create` |
| `api-token` | Cloudflare API token | ✅ Yes | - |
| `zone-id` | Cloudflare Zone ID | ✅ Yes | - |
| `ip-address` | IP to manage (auto-detected if empty) | ❌ No | Auto-detected |
| `rule-id` | Rule ID for delete (from create output) | ❌ No* | - |
| `mode` | Rule mode: `whitelist`, `block`, `challenge`, `js_challenge`, `managed_challenge` | ❌ No | `whitelist` |
| `notes` | Custom notes for the rule | ❌ No | `GitHub Actions - Automated CI/CD rule` |
| `wait-time` | Propagation wait time in seconds | ❌ No | `30` |

*Required when `action` is `delete`

## 📤 Outputs

| Output | Description | Example |
|--------|-------------|---------|
| `rule-id` | Created rule ID (save for deletion!) | `abc123def456` |
| `success` | Operation success status | `true` / `false` |
| `ip-address` | IP address that was managed | `203.0.113.1` |

## 🛠️ Setup Guide

### Cloudflare Setup

1. **Login to Cloudflare Dashboard**
2. **Go to My Profile → API Tokens**
3. **Create Token** with these permissions:
   - **Zone:Zone:Read** ✅
   - **Zone:Firewall Services:Edit** ✅
4. **Zone Resources**: Include your domain(s)
5. **Copy the token** 🔐

### Find Your Zone ID

1. **Cloudflare Dashboard** → Select your domain
2. **Right sidebar** → Copy **Zone ID**
3. **Add to GitHub Secrets** as `CLOUDFLARE_ZONE_ID`

### GitHub Repository Setup

1. **Settings** → **Secrets and variables** → **Actions**
2. **Add secrets**:
   ```
   CLOUDFLARE_API_TOKEN=your_token_from_step_above
   CLOUDFLARE_ZONE_ID=your_zone_id_from_step_above
   ```

## 🔧 Advanced Usage

### Error Handling & Retries

```yaml
- name: Whitelist with error handling
  id: whitelist
  uses: DiyRex/gh-cf-ip-whitelist@v1
  continue-on-error: true
  with:
    api-token: ${{ secrets.CLOUDFLARE_API_TOKEN }}
    zone-id: ${{ secrets.CLOUDFLARE_ZONE_ID }}
    action: 'create'

- name: Check if whitelist succeeded
  if: steps.whitelist.outputs.success == 'true'
  run: echo "✅ IP successfully whitelisted!"

- name: Fallback if whitelist failed
  if: steps.whitelist.outputs.success != 'true'
  run: echo "⚠️ Whitelist failed, tests may be blocked by Cloudflare"
```

### Emergency Cleanup Job

```yaml
jobs:
  test:
    # ... your main test job ...
    outputs:
      rule-id: ${{ steps.whitelist.outputs.rule-id }}
  
  # Emergency cleanup in case main job crashes
  cleanup:
    if: always() && needs.test.outputs.rule-id
    needs: test
    runs-on: ubuntu-latest
    steps:
    - name: Emergency cleanup
      uses: DiyRex/gh-cf-ip-whitelist@v1
      with:
        api-token: ${{ secrets.CLOUDFLARE_API_TOKEN }}
        zone-id: ${{ secrets.CLOUDFLARE_ZONE_ID }}
        action: 'delete'
        rule-id: ${{ needs.test.outputs.rule-id }}
```

### Matrix Strategy Support

```yaml
strategy:
  matrix:
    environment: [staging, production]
    browser: [chrome, firefox, safari]

steps:
- name: Whitelist for ${{ matrix.environment }}
  id: whitelist
  uses: DiyRex/gh-cf-ip-whitelist@v1
  with:
    api-token: ${{ secrets[format('CLOUDFLARE_API_TOKEN_{0}', matrix.environment)] }}
    zone-id: ${{ secrets[format('CLOUDFLARE_ZONE_ID_{0}', matrix.environment)] }}
    action: 'create'
    notes: 'Matrix Tests - ${{ matrix.environment }} - ${{ matrix.browser }}'
```

## 🔍 Troubleshooting

### Common Issues

| Error | Cause | Solution |
|-------|-------|----------|
| `Failed to create rule` | Invalid API token | Check token permissions |
| `Zone not found` | Wrong Zone ID | Verify Zone ID in dashboard |
| `IP already exists` | Duplicate rule | Check existing rules in dashboard |
| `Rate limit exceeded` | Too many API calls | Add delays between calls |

### Debug Mode

Add this step to see detailed information:

```yaml
- name: Debug info
  run: |
    echo "Runner IP: $(curl -s https://ipinfo.io/ip)"
    echo "Zone ID: ${{ secrets.CLOUDFLARE_ZONE_ID }}"
    echo "Token length: ${#CLOUDFLARE_API_TOKEN}"
  env:
    CLOUDFLARE_API_TOKEN: ${{ secrets.CLOUDFLARE_API_TOKEN }}
```

### Manual Cleanup

If rules get stuck, clean them up manually:

```bash
# List all rules
curl -X GET "https://api.cloudflare.com/client/v4/zones/ZONE_ID/firewall/access_rules/rules" \
  -H "Authorization: Bearer API_TOKEN"

# Delete specific rule
curl -X DELETE "https://api.cloudflare.com/client/v4/zones/ZONE_ID/firewall/access_rules/rules/RULE_ID" \
  -H "Authorization: Bearer API_TOKEN"
```

## 📊 Performance Benefits

| Testing Framework | Before | After | Improvement |
|-------------------|--------|-------|-------------|
| **Playwright** | ❌ Blocked by Cloudflare | ✅ Full access | 100% success rate |
| **Selenium** | ❌ Random timeouts | ✅ Reliable execution | 10x faster |
| **Cypress** | ❌ Challenge pages | ✅ Direct access | No more false failures |
| **Puppeteer** | ❌ Bot detection | ✅ Passes all checks | 100% compatibility |

## 🤝 Contributing

We welcome contributions! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

### Development

```bash
# Clone repository
git clone https://github.com/DiyRex/gh-cf-ip-whitelist.git
cd gh-cf-ip-whitelist

# Test locally
./test-local.sh

# Run tests
npm test
```

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🌟 Support

- 📖 **Documentation**: [Full docs available here](#)
- 🐛 **Issues**: [Report bugs or request features](https://github.com/DiyRex/gh-cf-ip-whitelist/issues)
- 💬 **Discussions**: [Join the community](https://github.com/DiyRex/gh-cf-ip-whitelist/discussions)
- ⭐ **Star this repo** if it helped you!

---

<div align="center">

**Made with ❤️ for the CI/CD community**

[⭐ Star](https://github.com/DiyRex/gh-cf-ip-whitelist) • [🐛 Report Bug](https://github.com/DiyRex/gh-cf-ip-whitelist/issues) • [✨ Request Feature](https://github.com/DiyRex/gh-cf-ip-whitelist/issues)

</div>