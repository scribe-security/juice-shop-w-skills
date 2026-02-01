# Proposed Minimal Dependency Upgrades

## Overview

This document proposes **minimal, low-risk upgrades** to address dependency vulnerabilities identified in the vulnerability triage report. The focus is on:

1. **Non-breaking changes** that can be safely applied
2. **Addressing high/critical vulnerabilities** where possible
3. **Respecting intentionally vulnerable packages** for training purposes
4. **Automated fixes** via `npm audit fix` for transitive dependencies

---

## Phase 1: Automated Non-Breaking Fixes (RECOMMENDED)

### Action: Run `npm audit fix`

This command will automatically update transitive dependencies to non-vulnerable versions without breaking changes.

**Expected to fix:**
- `diff` vulnerability (transitive via mocha)
- `tar` vulnerabilities (partial, transitive)
- Other minor transitive dependency updates

**Commands:**
```bash
# Root package
npm audit fix

# Frontend package
cd frontend && npm audit fix
```

**Risk Level:** ⭐ Very Low (automated, non-breaking)

**Testing Required:**
- Run lint: `npm run lint`
- Run tests: `npm test && npm run frisby`
- Run frontend tests: `cd frontend && npm test`

---

## Phase 2: Direct Dependency Upgrades (LOW RISK)

### 2.1 js-yaml: 3.14.0 → 3.14.2

**Vulnerability:** Prototype pollution (GHSA-mh29-5h37-fv8m)
**Severity:** Moderate
**Type:** Direct dependency
**Breaking:** No (patch version)

**Command:**
```bash
npm install js-yaml@^3.14.2
```

**Risk Level:** ⭐⭐ Low (patch version, well-tested)

---

## Phase 3: Selective Breaking Changes (MEDIUM RISK)

These upgrades require testing but address significant vulnerabilities.

### 3.1 check-dependencies: 1.1.1 → 2.0.0+

**Vulnerability:** Uncontrolled resource consumption via braces (GHSA-grv7-fg5c-xmjg)
**Severity:** High
**Type:** Direct dependency

**Command:**
```bash
npm install check-dependencies@^2.0.0
```

**Risk Level:** ⭐⭐⭐ Medium (major version, but stable API)

**Testing Focus:**
- Verify dependency checking still works during `npm install`

---

### 3.2 mocha: 8.4.0 → 11.7.5+

**Vulnerabilities:** 
- minimatch ReDoS (GHSA-f8q6-p94x-37v3)
- nanoid predictability (GHSA-mwcw-c2x4-8c55, GHSA-qrpm-p2h7-hrv2)
**Severity:** High (minimatch), Moderate (nanoid)
**Type:** Direct dev dependency

**Command:**
```bash
npm install --save-dev mocha@^11.7.5
```

**Risk Level:** ⭐⭐⭐ Medium (major version, test framework)

**Testing Focus:**
- Run server tests: `npm run test:server`
- Verify test output format hasn't changed

---

### 3.3 eslint: 8.57.1 → 9.39.2+

**Vulnerability:** Stack overflow with circular references (GHSA-p5wg-g6qr-c7cg)
**Severity:** Moderate
**Type:** Direct dev dependency

**Command:**
```bash
npm install --save-dev eslint@^9.39.2
npm install --save-dev @typescript-eslint/eslint-plugin@latest @typescript-eslint/parser@latest
```

**Risk Level:** ⭐⭐⭐⭐ Medium-High (major version, requires config updates)

**Additional Steps:**
- May require updating eslint config
- Update typescript-eslint plugins to compatible versions

**Testing Focus:**
- Run lint: `npm run lint`
- Check for new linting errors

---

## Phase 4: Frontend Upgrades (RECOMMENDED FOR SECURITY)

### 4.1 Frontend: socket.io-client: 3.1.0 → 4.8.3

**Vulnerabilities:**
- ws DoS (GHSA-3h5v-q93c-6h6q)
- parseuri ReDoS (GHSA-6fx8-h7jm-663j)
- socket.io-parser validation (GHSA-cqmj-92xf-r6r9)
**Severity:** High (ws), Moderate (others)

**Command:**
```bash
cd frontend
npm install socket.io-client@^4.8.3
```

**Risk Level:** ⭐⭐⭐⭐ Medium (major version, may affect real-time features)

**Testing Focus:**
- Test real-time chat features
- Verify WebSocket connections
- Check scoreboard updates

---

### 4.2 Backend: socket.io: 3.1.2 → 4.8.3

**Vulnerabilities:** (same as 4.1 above)

**Command:**
```bash
npm install socket.io@^4.8.3
```

**Risk Level:** ⭐⭐⭐⭐ Medium (must coordinate with frontend upgrade)

**Note:** This should be done **in tandem** with the frontend socket.io-client upgrade.

---

## Packages NOT Recommended for Upgrade

### Intentionally Vulnerable (Per .dependabot/config.yml)

These packages are **part of the training curriculum** and should remain vulnerable:

1. **express-jwt@0.1.3** - Do not upgrade
2. **jsonwebtoken@0.4.0** - Do not upgrade
3. **sanitize-html@1.4.2** - Do not upgrade
4. **unzipper@0.9.15** - Do not upgrade

### No Fix Available

1. **marsdb** - Command injection (GHSA-5mrr-rgp6-x4gr)
   - Consider migration to alternative if possible
   - Otherwise, document compensating controls

2. **notevil** - Sandbox escape (GHSA-8g4m-cjm2-96wq)
   - Review usage and consider alternatives
   - Document risk acceptance if needed

3. **lodash.set** (via grunt-replace-json) - Prototype pollution
   - Check if grunt-replace-json can be replaced

4. **vm2** (via juicy-chat-bot) - Sandbox escape vulnerabilities
   - Option: Downgrade juicy-chat-bot to 0.6.4
   - Risk: May break chat functionality

---

## Recommended Implementation Order

### Immediate (Next Release)
1. ✅ Phase 1: Run `npm audit fix` (both root and frontend)
2. ✅ Phase 2: Upgrade js-yaml to 3.14.2

### Short-term (Within 1-2 Releases)
3. Phase 3.1: Upgrade check-dependencies to 2.0.0
4. Phase 4.1 + 4.2: Upgrade socket.io packages (coordinated)

### Medium-term (Within 3-6 Releases)
5. Phase 3.2: Upgrade mocha to 11.7.5
6. Phase 3.3: Upgrade eslint to 9.39.2

### Long-term (Evaluate Separately)
7. Consider alternatives for marsdb, notevil
8. Evaluate juicy-chat-bot downgrade or vm2 mitigation

---

## Validation Checklist

After each upgrade phase:

- [ ] Code lints without errors: `npm run lint`
- [ ] All unit tests pass: `npm test`
- [ ] API tests pass: `npm run frisby`
- [ ] Frontend tests pass: `cd frontend && npm test`
- [ ] E2E tests pass: `npm run cypress:open`
- [ ] RSN passes (if challenge code modified): `npm run rsn`
- [ ] Application starts: `npm start`
- [ ] Key features work (login, search, checkout, admin)

---

## Risk Mitigation

- **Create feature branch** for each phase
- **Run full test suite** before merging
- **Document any behavior changes**
- **Monitor for regressions** post-deployment
- **Keep rollback plan** ready

---

## Summary of Changes

| Package | Current | Proposed | Severity Fixed | Risk Level |
|---------|---------|----------|----------------|------------|
| (transitive) | various | various | Mixed | ⭐ Very Low |
| js-yaml | 3.14.0 | 3.14.2 | Moderate | ⭐⭐ Low |
| check-dependencies | 1.1.1 | 2.0.0 | High | ⭐⭐⭐ Medium |
| mocha | 8.4.0 | 11.7.5 | High | ⭐⭐⭐ Medium |
| eslint | 8.57.1 | 9.39.2 | Moderate | ⭐⭐⭐⭐ Med-High |
| socket.io | 3.1.2 | 4.8.3 | High | ⭐⭐⭐⭐ Medium |
| socket.io-client | 3.1.0 | 4.8.3 | High | ⭐⭐⭐⭐ Medium |

**Total High/Critical Addressed:** 5 vulnerability classes
**Breaking Changes:** 5 (phases 3 & 4)
**Non-Breaking Changes:** 2 (phases 1 & 2)

---

## Consultation Required

Before implementing Phase 3 and 4 changes:
1. Confirm breaking changes are acceptable for current release cycle
2. Verify test infrastructure is ready
3. Ensure adequate time for regression testing
4. Review if any intentional vulnerabilities will be affected

---

**Document Created:** 2026-02-01
**Next Review:** After Phase 1 & 2 implementation
