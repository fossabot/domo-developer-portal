---
stoplight-id: e947d87e17547
---

# ryuu.js

> A powerful JavaScript SDK for building custom applications within the Domo platform

[![npm version](https://img.shields.io/npm/v/ryuu.js.svg)](https://www.npmjs.com/package/ryuu.js)
[![License](https://img.shields.io/badge/license-SEE%20LICENSE-blue.svg)](./LICENSE)
[![TypeScript](https://img.shields.io/badge/TypeScript-Ready-blue.svg)](https://www.typescriptlang.org/)

ryuu.js (published as `ryuu.js`, developed as `domo.js`) is a comprehensive JavaScript library that enables developers to build interactive custom applications within the Domo platform. It provides seamless communication between your custom app and the Domo environment, supporting data fetching, real-time events, filters, variables, and cross-platform mobile integration.

---

## Choose Your Version

Select the documentation version that matches your ryuu.js installation:

### [v5+ Documentation](./domo.js-v5.md) ⭐ Latest (Recommended)

**Current version with latest features and improvements**

- MessageChannel API for improved communication
- Enhanced request tracking with ASK-ACK-REPLY pattern
- `Domo.navigate()` method for programmatic navigation
- `Domo.extend()` for customizing behavior
- Modern Fetch API support
- Improved TypeScript definitions
- Better mobile platform support (iOS and Android)

**Latest Version:** v5.x

[View v5+ Documentation →](./domo.js-v5.md)

---

### [v4 and Earlier Documentation](./domo.js-v4.md) 📦 Legacy

**For apps using ryuu.js v4.x or earlier**

- Traditional postMessage-based communication
- Basic iOS and Android mobile support
- Core HTTP methods (get, post, put, delete)
- Event listeners for data, filters, and app data
- Filter and variable management

**Supported Versions:** v4.x and earlier

[View v4 and Earlier Documentation →](./domo.js-v4.md)

---

## Installation

### NPM

```bash
# Install latest version (v5+)
npm install ryuu.js

# Install specific v4 version
npm install ryuu.js@4
```

### CDN / Script Tag

```html
<!-- Latest version (v5+) -->
<script src="https://unpkg.com/ryuu.js"></script>

<!-- Specific v4 version -->
<script src="https://unpkg.com/ryuu.js@4"></script>
```

---

## Which Version Should I Use?

### Use v5+ if:
- Starting a new project
- Want the latest features and improvements
- Need better mobile platform support
- Want request tracking and better error handling
- Prefer modern JavaScript patterns

### Use v4 if:
- Maintaining an existing app built on v4
- Need to match legacy behavior
- Not ready to migrate to v5+

**Note:** v4 methods still work in v5+ with deprecation warnings. See the [v5+ Migration Guide](./domo.js-v5.md#migration-guide) for upgrade instructions.

---

## Quick Links

- **npm package:** [ryuu.js on npm](https://www.npmjs.com/package/ryuu.js)
- **GitHub Issues:** Report issues and contribute
- **Domo Developer Portal:** Visit for more developer resources

---

## Support

For issues, questions, or contributions:

- **Issues:** Open an issue on GitHub
- **Domo Developer Portal:** Visit the Domo developer documentation
- **Community Forums:** Connect with other developers

---

**Made with ❤️ by the Domo team**
