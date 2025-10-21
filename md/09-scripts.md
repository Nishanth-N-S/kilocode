# Scripts Directory (scripts/) Documentation

The `scripts/` directory contains build, deployment, and utility scripts for the Kilo Code repository.

## Directory Structure

```
scripts/
├── kilocode/              # Kilo-specific scripts
│   └── server/            # Server-related scripts
├── bootstrap.mjs          # Bootstrap/initialization script
├── install-vsix.js        # Install VSIX package locally
├── update-contributors.js # Update contributors list
└── link-packages.ts       # Link/unlink workspace packages
```

## Main Scripts

### `bootstrap.mjs`

**Purpose**: Initialize and bootstrap the monorepo

**What it does**:
1. Check Node.js version
2. Install pnpm if needed
3. Initialize git submodules
4. Install dependencies
5. Set up development environment

**Usage**:
```bash
node scripts/bootstrap.mjs
```

**When it runs**:
- Automatically on `pnpm install`
- Via `preinstall` and `install` npm scripts
- Can be run manually

**Features**:
- Version validation
- Dependency checking
- Git submodule initialization
- Error handling and reporting

**Code Example**:
```javascript
// Check Node.js version
const nodeVersion = process.version
const requiredVersion = '20.19.2'

if (!nodeVersion.startsWith('v20')) {
  console.error(`Node.js ${requiredVersion} required`)
  process.exit(1)
}

// Initialize submodules
execSync('git submodule update --init --recursive')
```

---

### `install-vsix.js`

**Purpose**: Install built VSIX package to VS Code

**What it does**:
1. Find latest VSIX file in `bin/`
2. Uninstall previous version
3. Install new VSIX
4. Confirm installation

**Usage**:
```bash
node scripts/install-vsix.js

# Or via npm script
pnpm install:vsix
```

**Features**:
- Automatic VSIX detection
- Version management
- Error handling
- Installation confirmation

**Code Example**:
```javascript
const vsixPath = findLatestVsix('bin/')
const extensionId = 'kilocode.kilo-code'

// Uninstall old version
execSync(`code --uninstall-extension ${extensionId}`)

// Install new version
execSync(`code --install-extension ${vsixPath}`)
```

---

### `update-contributors.js`

**Purpose**: Update contributors list in README

**What it does**:
1. Fetch contributors from GitHub API
2. Generate contributor HTML
3. Update README.md
4. Commit changes (optional)

**Usage**:
```bash
node scripts/update-contributors.js

# Or via npm script
pnpm update-contributors
```

**Features**:
- GitHub API integration
- Avatar images
- Automatic README update
- Customizable template

**API Usage**:
```javascript
const response = await fetch(
  'https://api.github.com/repos/Kilo-Org/kilocode/contributors'
)
const contributors = await response.json()
```

**Template**:
```javascript
const generateHTML = (contributors) => `
<table>
  <tr>
    ${contributors.map(c => `
      <td align="center">
        <a href="${c.html_url}">
          <img src="${c.avatar_url}" width="100" />
          <br />${c.login}
        </a>
      </td>
    `).join('')}
  </tr>
</table>
`
```

---

### `link-packages.ts`

**Purpose**: Link or unlink workspace packages

**What it does**:
1. Find all workspace packages
2. Create symlinks for local development
3. Unlink when needed

**Usage**:
```bash
# Link packages
pnpm link-workspace-packages

# Unlink packages
pnpm unlink-workspace-packages
```

**Features**:
- Automatic package discovery
- Symlink management
- Cross-platform support
- Error handling

**Code Example**:
```typescript
const packages = await findWorkspacePackages()

for (const pkg of packages) {
  if (options.unlink) {
    await unlinkPackage(pkg)
  } else {
    await linkPackage(pkg)
  }
}
```

---

## Kilo-Specific Scripts

### `scripts/kilocode/`

**Purpose**: Scripts specific to Kilo Code features

#### `server/`

**Purpose**: Server-related scripts and utilities

**Contents**:
- Server startup scripts
- Configuration helpers
- Deployment scripts

**Documentation**: See `scripts/kilocode/server/README.md`

---

## Common Script Patterns

### Error Handling

```javascript
try {
  // Script logic
  console.log('Success!')
} catch (error) {
  console.error('Error:', error.message)
  process.exit(1)
}
```

### Command Execution

```javascript
import { execSync } from 'child_process'

const result = execSync('command', {
  encoding: 'utf-8',
  stdio: 'inherit' // Show output
})
```

### File Operations

```javascript
import { readFileSync, writeFileSync } from 'fs'

const content = readFileSync('file.txt', 'utf-8')
const updated = content.replace(/old/g, 'new')
writeFileSync('file.txt', updated)
```

### Path Resolution

```javascript
import { resolve, join } from 'path'
import { fileURLToPath } from 'url'

const __dirname = fileURLToPath(new URL('.', import.meta.url))
const projectRoot = resolve(__dirname, '..')
```

---

## Package.json Scripts

Scripts are typically called via npm/pnpm scripts:

```json
{
  "scripts": {
    "preinstall": "node scripts/bootstrap.mjs",
    "install": "node scripts/bootstrap.mjs",
    "install:vsix": "node scripts/install-vsix.js",
    "update-contributors": "node scripts/update-contributors.js",
    "link-workspace-packages": "tsx scripts/link-packages.ts",
    "unlink-workspace-packages": "tsx scripts/link-packages.ts --unlink"
  }
}
```

---

## Script Development

### TypeScript Scripts

Use `tsx` for TypeScript execution:

```typescript
// script.ts
import { someFunction } from './utils'

async function main() {
  // Script logic
}

main().catch(console.error)
```

**Run**:
```bash
tsx scripts/script.ts
```

### JavaScript Scripts

Use modern JavaScript:

```javascript
// script.mjs
import { someFunction } from './utils.mjs'

async function main() {
  // Script logic
}

main().catch(console.error)
```

**Run**:
```bash
node scripts/script.mjs
```

---

## Best Practices

1. **Error Handling**: Always handle errors gracefully
2. **Logging**: Provide clear, informative output
3. **Exit Codes**: Use proper exit codes (0 for success, non-zero for errors)
4. **Documentation**: Document what the script does and how to use it
5. **Cross-Platform**: Ensure scripts work on Windows, macOS, and Linux
6. **Idempotency**: Scripts should be safe to run multiple times
7. **Dependencies**: Minimize external dependencies
8. **Testing**: Test scripts in different environments

---

## Adding New Scripts

### 1. Create Script File

```bash
touch scripts/my-script.mjs
chmod +x scripts/my-script.mjs
```

### 2. Add Shebang (optional)

```javascript
#!/usr/bin/env node

// Script code
```

### 3. Add npm Script

```json
{
  "scripts": {
    "my-script": "node scripts/my-script.mjs"
  }
}
```

### 4. Document

Add to this documentation and include inline comments.

### 5. Test

Test on all platforms:
- Windows
- macOS (Intel and Apple Silicon)
- Linux

---

## CI/CD Integration

Scripts are used in GitHub Actions:

```yaml
# .github/workflows/build.yml
- name: Bootstrap
  run: pnpm install

- name: Build
  run: pnpm build

- name: Install VSIX
  run: pnpm install:vsix
```

---

## Troubleshooting

### Permission Denied

**Problem**: Script not executable

**Solution**:
```bash
chmod +x scripts/script.mjs
```

### Module Not Found

**Problem**: Import path incorrect

**Solution**:
- Use `.mjs` extension for ES modules
- Check import paths
- Ensure file exists

### Command Not Found

**Problem**: Command in `execSync` not available

**Solution**:
- Check if command is installed
- Add to PATH
- Use full path to command

---

*Last updated: 2025-10-21*
