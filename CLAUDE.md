# Claude Instructions for bus_app

## Versioning
- The app version is defined as `const APP_VERSION = 'vX.Y.Z'` in `bus_app/index.html`
- **Increment the minor version (Y) by 1** for every change that is NOT a bug fix. Small tweaks should not lead to an increment 
- Bug fixes (regressions, incorrect behaviour) do **not** bump the version
- Example: v1.0.0 → v1.1.0 when adding a new feature or behaviour change
- Please also update the README.md file if it requires updating

## Git
- **Do not offer to push to git** — the user handles all git commits and pushes themselves

## Self-Testing
- After writing or modifying code, **always attempt to verify the change works** where possible:
  - For JavaScript logic, run a quick Node.js snippet via Bash to validate the logic
  - Check for syntax errors by running `node --check` on extracted script sections
  - For HTML/CSS changes, verify the structure is valid (no unclosed tags, correct nesting)
  - Flag anything that cannot be tested in the VM environment and explain why
