# Upstream Sync Status

This document tracks the integration points between the upstream MCP Inspector and our assessment enhancements to facilitate future upstream syncs.

## Sync Information

| Field                   | Value                                             |
| ----------------------- | ------------------------------------------------- |
| **Upstream Repository** | https://github.com/modelcontextprotocol/inspector |
| **Last Sync Version**   | v0.21.1                                           |
| **Last Sync Date**      | 2026-03-22                                        |
| **Last Sync Commit**    | `7c8b031ffac663031ee5a0d0a7dc79735321c2d1`        |

## Integration Points

### Architecture: CLI-First (v0.21.1+)

As of the v0.21.1 sync (2026-03-22), **App.tsx has zero assessment integration**. The GUI assessment tab was removed — upstream App.tsx is accepted wholesale during syncs. The CLI (`cli/src/`) is now the sole assessment interface.

**No upstream files require modification for assessment integration.** All assessment code lives in fork-only files (listed below) with zero overlap with upstream.

### Historical Note

Prior to v0.21.1, App.tsx had 7 integration points for a GUI assessment tab. These were removed because:

- The CLI is the published npm interface (`@bryan-thompson/inspector-assessment`)
- Accepting upstream App.tsx wholesale eliminates all merge conflicts
- The GUI tab was not actively maintained

## Files Added (No Conflict Risk)

These files are entirely new and have no upstream equivalents:

### Assessment Core (`client/src/services/assessment/`)

- `AssessmentOrchestrator.ts`
- `TestScenarioEngine.ts`
- `ResponseValidator.ts`
- `TestDataGenerator.ts`
- `ToolClassifier.ts`
- `PolicyComplianceGenerator.ts`
- All files in `modules/`
- All files in `__tests__/`

### Assessment Libraries (`client/src/lib/`)

- `assessmentTypes.ts`
- `assessmentDiffer.ts`
- `aupPatterns.ts`
- `prohibitedLibraries.ts`
- `securityPatterns.ts`
- `policyMapping.ts`
- `moduleScoring.ts`
- `distributionDetection.ts`
- `reportFormatters/`

### Assessment Components (`client/src/components/`)

- `AssessmentTab.tsx`
- `ExtendedAssessmentCategories.tsx`
- `ReviewerAssessmentView.tsx`
- `UnifiedAssessmentHeader.tsx`
- `AssessmentCategoryFilter.tsx`
- `AssessmentSummary.tsx`
- `AssessmentChecklist.tsx`
- `ui/tool-selector.tsx`
- `ui/badge.tsx`
- `ui/progress.tsx`

### CLI Additions (`cli/src/`)

- `assess-full.ts`
- `assess-security.ts`
- `assessmentState.ts`
- `validate-testbed.ts`

### Scripts (`scripts/`)

- `run-full-assessment.ts`
- `run-security-assessment.ts`
- `lib/jsonl-events.ts`

### Documentation (`docs/`)

- All files in this directory

## Sync Procedure

### Automated Sync (Recommended)

Use the sync helper script for guided upstream syncing:

```bash
# Check status and view changes (safe, read-only)
npm run sync:upstream

# Or run individual commands:
./scripts/sync-upstream.sh status    # Show sync status
./scripts/sync-upstream.sh diff      # View upstream changes to App.tsx
./scripts/sync-upstream.sh merge     # Attempt merge with conflict guidance
./scripts/sync-upstream.sh validate  # Build and test after merge
```

The script automatically:

- Fetches upstream and shows divergence
- Highlights if integration lines are affected
- Provides merge conflict resolution guidance
- Prompts to update this document after successful merge

### Manual Sync

#### Before Syncing

1. **Check upstream changes to App.tsx**:

   ```bash
   git fetch upstream
   git diff upstream/main...HEAD -- client/src/App.tsx
   ```

2. **Identify conflicts in integration lines**:
   ```bash
   git diff fe393e514a..upstream/main -- client/src/App.tsx
   ```

#### During Sync

1. **Merge upstream**:

   ```bash
   git fetch upstream
   git merge upstream/main
   ```

2. **If App.tsx conflicts**, manually apply the 6 integration points listed above

3. **Verify integration**:
   ```bash
   npm run build
   npm test
   ```

#### After Sync

1. **Update this document** with:
   - New sync version
   - New sync date
   - New sync commit
   - Any line number changes

2. **Test assessment functionality**:
   ```bash
   npm run assess -- --server <server-name> --config <config.json>
   ```

## Risk Assessment

| Area                 | Risk | Notes                                             |
| -------------------- | ---- | ------------------------------------------------- |
| App.tsx              | None | Accepted wholesale from upstream, zero changes    |
| client/src/services/ | Low  | Assessment modules are fork-only, no overlap      |
| cli/src/             | None | Entirely fork-only, no upstream equivalent        |
| package.json files   | Low  | Fork metadata (name, version) must be kept        |
| client/lib/ types    | Low  | Upstream SDK type changes may need `as any` casts |

## Future Improvements

To further reduce sync friction:

1. **Plugin architecture**: Enable assessment as optional npm package

---

_Last updated: 2026-03-22_
