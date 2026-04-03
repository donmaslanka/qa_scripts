# RESPONSE-Katalon-QA Jenkins Pipeline

## Correct katalonc command

```bash
katalonc \
  -noSplash \
  -runMode=console \
  -projectPath="$WORKSPACE/RESPONSE.prj" \
  -testSuiteCollectionPath="Test Suites/Headless-QA" \
  -apiKey="$KATALON_API_KEY" \
  -orgID="2333388" \
  -retry=0
```

## Key flags explained

| Flag | Value | Why |
|---|---|---|
| `-testSuiteCollectionPath` | `Test Suites/Headless-QA` | Headless-QA is a **Collection** not a single suite |
| `-orgID` | `2333388` | LeadFusion org ID |
| NO `-browserType` | — | Browser defined per-suite inside Headless-QA.ts |
| NO `--config` | — | Causes Eclipse workspace conflict |

## katalonc.ini on ec2-agent-01

Location: `/opt/katalon/katalonc.ini`

```
-Xms128m
-Xmx512m
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200
-XX:+UseStringDeduplication
-XX:+ExitOnOutOfMemoryError
-Djava.awt.headless=true
```

## Case-sensitive file fixes (Linux vs Windows)

These renames must always be applied when syncing from client:

| Wrong (Windows) | Correct (Linux) | Applies to |
|---|---|---|
| `savings.ts` | `Savings.ts` | Test Suites/ |
| `Hsa.ts` | `HSA.ts` | Test Suites/ |
| `Creditline.ts` | `CreditLine.ts` | Test Suites/ |
| `Test Cases/Card/` | `Test Cases/CreditCard/` | Test Cases/ + Scripts/ |
| `Test Cases/Hsa/` | `Test Cases/HSA/` | Test Cases/ + Scripts/ |
| `Test Cases/Creditline/` | `Test Cases/CreditLine/` | Test Cases/ + Scripts/ |

Also fix XML refs inside CreditCard.ts and CreditLine.ts:
- `Test Cases/Card/` → `Test Cases/CreditCard/`
- `Test Cases/Creditline/` → `Test Cases/CreditLine/`

## Syncing client updates

Use `sync_updated_tests.ps1` — it applies all fixes automatically.
NEVER push directly to main. Use a feature branch + PR.
