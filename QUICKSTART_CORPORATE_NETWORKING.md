# Corporate Networking Quick Start Guide

This guide helps you quickly set up OpenCode to work with corporate proxies and custom CA certificates.

## 🚀 Quick Installation

### Step 1: Install Dependencies

```bash
cd /home/user/opencode
bun install
```

This installs `undici@6.21.0` and all required dependencies.

### Step 2: Set Environment Variables

```bash
# Required: Your API key
export CORPORATE_AI_KEY=your-api-key-here

# Optional: Proxy settings (auto-detected when using proxy: "auto")
export HTTPS_PROXY=http://proxy.company.com:8080
export HTTP_PROXY=http://proxy.company.com:8080
export NO_PROXY=localhost,127.0.0.1,.local

# Optional: Proxy authentication (or let it prompt interactively)
export PROXY_USER=your.username
export PROXY_PASS=your-password

# Optional: CA certificates (or specify in config file)
export NODE_EXTRA_CA_CERTS=/etc/ssl/certs/corporate-ca-bundle.crt
```

### Step 3: Create Configuration File

Create `.opencode/config.json` or `opencode.jsonc` in your project:

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "corporate-ai": {
      "npm": "@ai-sdk/openai-compatible",
      "api": "https://ai.internal.company.com/v1",
      "options": {
        "apiKey": "{env:CORPORATE_AI_KEY}",
        "proxy": "auto",
        "tls": {
          "ca": "/etc/ssl/certs/corporate-ca-bundle.crt"
        }
      },
      "models": {
        "gpt-4": {
          "name": "Corporate GPT-4",
          "limit": {
            "context": 128000,
            "output": 4096
          }
        }
      }
    }
  }
}
```

### Step 4: Test Connection

```bash
./packages/opencode/bin/opencode chat --provider corporate-ai
```

## 📋 What You'll See

When you connect, the system will:

```
🔍 Detected proxy from environment: http://proxy.company.com:8080
📜 Loading CA certificate bundle: /etc/ssl/certs/corporate-ca-bundle.crt
   Found 3 certificates in bundle
🌐 Using proxy: http://***:***@proxy.company.com:8080

Connecting to https://ai.internal.company.com/v1...
```

If proxy authentication is needed:

```
🔐 Proxy authentication required
🔑 Please enter your proxy credentials

Proxy: http://proxy.company.com:8080
? Username: your.name
? Password: ******** (hidden)

✅ Proxy authentication successful
✅ Connected to Corporate AI
```

## 🎯 Common Configuration Patterns

### Pattern 1: Auto-Detect Everything

**Best for**: Standard corporate environments

```jsonc
{
  "provider": {
    "corporate-ai": {
      "npm": "@ai-sdk/openai-compatible",
      "api": "https://ai.company.com/v1",
      "options": {
        "apiKey": "{env:CORPORATE_AI_KEY}",
        "proxy": "auto",  // Auto-detect from environment
        "tls": {
          "ca": "{env:NODE_EXTRA_CA_CERTS}"  // Use standard env var
        }
      }
    }
  }
}
```

### Pattern 2: Explicit Configuration

**Best for**: Complex setups with multiple certificates

```jsonc
{
  "provider": {
    "corporate-ai": {
      "npm": "@ai-sdk/openai-compatible",
      "api": "https://ai.company.com/v1",
      "options": {
        "apiKey": "{env:CORPORATE_AI_KEY}",
        "proxy": "http://proxy.company.com:8080",
        "proxyAuth": {
          "username": "{env:PROXY_USER}",
          "password": "{env:PROXY_PASS}"
        },
        "tls": {
          "ca": [
            "/etc/ssl/certs/root-ca.crt",
            "/etc/ssl/certs/intermediate-ca.crt",
            "/etc/ssl/certs/company-ca.crt"
          ]
        }
      }
    }
  }
}
```

### Pattern 3: Client Certificate (Mutual TLS)

**Best for**: High-security environments

```jsonc
{
  "provider": {
    "secure-ai": {
      "npm": "@ai-sdk/openai-compatible",
      "api": "https://secure-ai.company.com/v1",
      "options": {
        "apiKey": "{env:AI_KEY}",
        "tls": {
          "ca": "/etc/ssl/certs/company-ca.crt",
          "cert": "/home/user/.ssl/client-cert.pem",
          "key": "/home/user/.ssl/client-key.pem",
          "rejectUnauthorized": true
        }
      }
    }
  }
}
```

## 🔍 Finding Your Corporate CA Certificates

### Windows
```powershell
# Common locations
C:\ProgramData\YourCompany\certificates\ca-bundle.crt
C:\Users\YourName\AppData\Local\certificates\

# Export from Certificate Store
certutil -store Root > ca-bundle.crt
```

### macOS
```bash
# Common locations
/usr/local/share/ca-certificates/
/etc/ssl/certs/

# Export from Keychain
security find-certificate -a -p > ca-bundle.crt
```

### Linux
```bash
# Common system locations
/etc/ssl/certs/ca-certificates.crt
/etc/pki/tls/certs/ca-bundle.crt
/etc/ssl/ca-bundle.pem

# Company-specific locations
/usr/local/share/ca-certificates/
/opt/company/certs/
```

## 🔧 Troubleshooting

### Certificate Errors

**Error**: `self-signed certificate in certificate chain`

**Solution**: Add your corporate CA certificate:
```jsonc
{
  "tls": {
    "ca": "/path/to/corporate-ca-bundle.crt"
  }
}
```

### Proxy Connection Fails

**Check proxy is reachable**:
```bash
curl -x http://proxy.company.com:8080 https://google.com
```

**Verify proxy URL format**:
```bash
# Correct formats:
http://proxy.company.com:8080
http://username:password@proxy.company.com:8080

# Must include http:// or https://
```

### Multiple Certificates in Bundle

**Question**: I have multiple certificates in my bundle. Which one do I need?

**Answer**: Use them all! The system automatically detects and uses the correct one:

```jsonc
{
  "tls": {
    "ca": "/etc/ssl/certs/corporate-ca-bundle.crt"  // Can contain 100+ certs
  }
}
```

Output: `"Found 3 certificates in bundle"` ✅

### Interactive Prompts Not Showing

**Cause**: Non-interactive environment

**Solution**: Provide credentials via environment variables:
```bash
export PROXY_USER=your.username
export PROXY_PASS=your-password
```

Then reference in config:
```jsonc
{
  "proxyAuth": {
    "username": "{env:PROXY_USER}",
    "password": "{env:PROXY_PASS}"
  }
}
```

## ✅ Verification Checklist

Before using, verify:

- [ ] Dependencies installed (`bun install` completed)
- [ ] Configuration file created (`.opencode/config.json`)
- [ ] API key set in environment variable
- [ ] Corporate CA certificate file exists and is readable
- [ ] Proxy URL is correct (if using proxy)
- [ ] Can reach API endpoint (test with curl)

## 📚 Configuration Reference

### Proxy Options

```typescript
{
  // Auto-detect from environment/system settings
  "proxy": "auto"

  // Explicit proxy URL
  "proxy": "http://proxy.company.com:8080"

  // Disable proxy explicitly
  "proxy": false
}
```

### Proxy Authentication

```typescript
{
  "proxyAuth": {
    "username": "your.username",           // Literal value
    "username": "{env:PROXY_USER}",        // From environment variable
    "password": "{env:PROXY_PASS}"
  }
}
```

### TLS/Certificate Options

```typescript
{
  "tls": {
    // Single CA certificate file (can contain multiple certs)
    "ca": "/etc/ssl/certs/corporate-ca-bundle.crt",

    // OR multiple CA certificate files
    "ca": [
      "/etc/ssl/certs/root-ca.crt",
      "/etc/ssl/certs/intermediate-ca.crt"
    ],

    // Client certificate (optional, for mutual TLS)
    "cert": "/path/to/client-cert.pem",
    "key": "/path/to/client-key.pem",

    // Certificate validation (default: true)
    "rejectUnauthorized": true
  }
}
```

## 🎯 One-Command Setup

Copy and customize this for your environment:

```bash
# Set all environment variables
export CORPORATE_AI_KEY=your-api-key
export HTTPS_PROXY=http://proxy.company.com:8080
export NODE_EXTRA_CA_CERTS=/etc/ssl/certs/corporate-ca-bundle.crt

# Create config file
mkdir -p .opencode
cat > .opencode/config.json <<'EOF'
{
  "provider": {
    "corporate-ai": {
      "npm": "@ai-sdk/openai-compatible",
      "api": "https://ai.internal.company.com/v1",
      "options": {
        "apiKey": "{env:CORPORATE_AI_KEY}",
        "proxy": "auto",
        "tls": {
          "ca": "{env:NODE_EXTRA_CA_CERTS}"
        }
      },
      "models": {
        "gpt-4": {
          "name": "Corporate GPT-4"
        }
      }
    }
  }
}
EOF

# Install dependencies
bun install

# Test connection
./packages/opencode/bin/opencode chat --provider corporate-ai
```

## 🚀 Advanced Features

### Session Credential Caching

Proxy credentials are cached in memory for 1 hour after successful authentication. This means:

- First connection: Prompts for credentials
- Subsequent connections: Uses cached credentials
- After 1 hour: Prompts again for security

### NO_PROXY Support

Specify domains to bypass the proxy:

```bash
export NO_PROXY=localhost,127.0.0.1,.internal,.local
```

The system automatically checks if your target URL should bypass the proxy.

### System Proxy Auto-Detection

When using `"proxy": "auto"`, the system checks (in order):

1. **Environment variables**: `HTTPS_PROXY`, `HTTP_PROXY`, `ALL_PROXY`
2. **Windows**: Internet Settings registry
3. **macOS**: System Preferences via `scutil --proxy`
4. **Linux**: GNOME/KDE settings via `gsettings`
5. **Interactive prompt**: Asks if you need a proxy

### Security Features

✅ **Credential Masking**: Proxy credentials shown as `***:***@proxy` in logs
✅ **Memory-Only Cache**: Credentials never written to disk
✅ **Session Timeout**: Credential cache expires after 1 hour
✅ **Environment Variables**: Sensitive data kept in env vars, not config files

## 📖 Additional Documentation

For more details, see:

- **`CORPORATE_PROXY_FEATURE_SUMMARY.md`** - Complete feature overview
- **`packages/opencode/src/provider/CORPORATE_INTEGRATION_EXAMPLE.md`** - Detailed integration guide
- **`packages/opencode/src/provider/corporate-fetch.ts`** - Implementation source code

## ❓ FAQ

**Q: Do I need to know which certificate in my bundle to use?**
A: No! Just point to the bundle file. The system loads all certificates and automatically selects the right one.

**Q: Can the system auto-detect my corporate proxy?**
A: Yes! Set `"proxy": "auto"` and it will check environment variables and system settings.

**Q: Will it prompt me for proxy credentials like a browser?**
A: Yes! If the proxy requires authentication, you'll get an interactive prompt for username and password.

**Q: Are my credentials stored securely?**
A: Yes! Credentials are cached in memory only (never written to disk) and expire after 1 hour.

**Q: Is this backward compatible with existing configurations?**
A: Yes! All new options are optional. Existing configurations continue to work without changes.

## 🎉 You're Ready!

The corporate networking feature is fully integrated and ready to use. If you encounter any issues, check the troubleshooting section or refer to the detailed documentation files.
