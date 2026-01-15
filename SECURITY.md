# Security Policy

## Overview

WitMani Game Animator is a Claude Code plugin that generates game character animations using AI services. This document describes the security measures in place and best practices for using this plugin safely.

## API Key Handling

### Storage

- API keys are stored in `~/.config/witmani/config`
- Config file permissions are set to `600` (owner read/write only)
- Keys are **never** hardcoded in plugin source code
- Keys are **never** logged or displayed in output

### Transmission

API keys are only transmitted to their respective providers:

| Key | Transmitted To |
|-----|----------------|
| `FAL_KEY` | `fal.run` (fal.ai API) |
| `GEMINI_API_KEY` | `generativelanguage.googleapis.com` (Google) |

Keys are transmitted over HTTPS only.

### Best Practices

1. **Rotate keys regularly** - Generate new API keys periodically
2. **Use separate keys** - Don't reuse keys from other applications
3. **Monitor usage** - Check your API provider dashboard for unusual activity
4. **Revoke compromised keys** - If you suspect a key is compromised, revoke it immediately

## Data Privacy

### What This Plugin Sends

- Text prompts (character description, action)
- Generated images (for video generation and background removal)

### What This Plugin Does NOT Send

- Your API keys (except to their respective providers)
- Personal information
- Local file paths
- System information

### Local Storage

Generated animations are stored locally in the `animations/` directory. This directory is excluded from git by default via `.gitignore`.

## External Services

This plugin relies on the following external services:

| Service | Purpose | Privacy Policy |
|---------|---------|----------------|
| fal.ai | Image & video generation | [fal.ai/privacy](https://fal.ai/privacy) |
| Google Gemini | Image & video generation | [Google Privacy](https://policies.google.com/privacy) |

## Reporting Security Issues

If you discover a security vulnerability, please report it responsibly:

1. **Do NOT** create a public GitHub issue
2. Email the maintainer directly (see repository for contact)
3. Provide detailed steps to reproduce
4. Allow reasonable time for a fix before public disclosure

## Security Checklist for Users

Before using this plugin:

- [ ] Review the source code in `commands/` directory
- [ ] Ensure you trust the AI providers you configure
- [ ] Keep your API keys secure and don't share them
- [ ] Regularly check for plugin updates

## Version History

| Version | Security Updates |
|---------|------------------|
| 1.0.0 | Initial release with secure key storage |
