# PnpmAuditAnalyzer reports only one advisory per package

Minimal project with a single direct dependency, `lodash@4.17.11` (7 known advisories).
The same `package.json` is used in `npm/` and `pnpm/`.

Environment: dependency-check-cli 13.0.0, Linux x64, Node 20.20.2, npm 10.8.2, pnpm 9.12.0 and 10.32.1.

## Steps

```
cd pnpm && pnpm install --lockfile-only && pnpm audit --prod --json > ../reports/pnpm-audit.json && cd ..
cd npm  && npm install --package-lock-only && npm audit --omit=dev --json > ../reports/npm-audit.json && cd ..

OPTS="--noupdate --format HTML --format JSON --disableOssIndex --disableRetireJs --disableYarnAudit --disableAssembly"
dependency-check.sh --project odc-pnpm-repro --scan pnpm --pnpm "$(command -v pnpm)" $OPTS --out reports/odc-pnpm --log odc-pnpm.log
dependency-check.sh --project odc-pnpm-repro --scan npm  --disableNodeAuditCache       $OPTS --out reports/odc-npm  --log odc-npm.log
```

## Result

| Variant      | advisories from audit | vulnerabilities in ODC report              |
|--------------|-----------------------|--------------------------------------------|
| npm          | 7                     | 7 (1 critical, 3 high, 3 moderate)         |
| pnpm 9.12.0  | 7                     | 1 (GHSA-29mw-wpgm-hmr9, moderate)          |
| pnpm 10.32.1 | 7                     | 1 (GHSA-29mw-wpgm-hmr9, moderate)          |

With pnpm the debug log contains this line 6 times (`reports/odc-pnpm-debug-excerpt.log`):

```
Attempted to add the same dependency as this, likely due to merging identical dependencies obtained from different modules
```

In the pnpm audit JSON, `findings[].paths` is `[]` (pnpm 9) or `[".>lodash"]` (pnpm 10), so `NpmAuditParser` never sets the version (it only does when a path equals `module_name`).
