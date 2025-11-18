# Pre-Migration Checklist

**Created:** November 16, 2025  
**Last Updated:** November 16, 2025  
**Status:** Pre-Migration

## Pre-Migration Checklist

Before starting the migration to the revamped UNITARES framework, complete these steps:

- [ ] **Backup current MCP server code**
  - Location: `/Users/cirwel/Library/Mobile Documents/iCloud~md~obsidian/Documents/governance-monitor-mcp/src/server.py`
  - Action: Create backup copy before making changes
  - Command: `cp server.py server.py.backup`

- [ ] **Backup existing CSV files**
  - Location: `~/Library/Mobile Documents/iCloud~md~obsidian/Documents/governance-monitor-mcp/data/`
  - Files: `governance_history_*.csv`
  - Action: Copy entire `data/` directory
  - Command: `cp -r data/ data_backup/`

- [ ] **Review current implementation**
  - Read: Current MCP server code
  - Understand: How V, I, E are currently calculated
  - Document: Current behavior and parameters
  - Note: Any customizations or special cases

- [ ] **Understand revamped framework**
  - Read: `docs/mcp/unitares-revamped-framework.md`
  - Read: `docs/mcp/void-metric-analysis.md`
  - Read: `docs/mcp/integrity-computation-comparison.md`
  - Understand: Key differences and improvements

- [ ] **Plan testing strategy**
  - Unit tests: Test each calculation separately
  - Integration tests: Test with actual CSV data
  - Regression tests: Verify existing functionality
  - Monitoring: Plan how to monitor after migration

- [ ] **Prepare rollback plan**
  - Backup: Code and data (see above)
  - Document: Current configuration
  - Plan: Steps to revert if needed
  - Test: Rollback procedure

## Quick Reference

**Key Files:**
- MCP Server: `governance-monitor-mcp/src/server.py`
- CSV Data: `governance-monitor-mcp/data/governance_history_*.csv`
- Revamped Framework: `scripts/unitares_revamped.py`
- Migration Guide: `docs/mcp/MCP_MIGRATION_GUIDE.md`

**Key Changes:**
- V: Pure integration → Leaky integrator (γ = 0.85)
- I: Integrated → Direct (I = 1 - S)
- Lambda: Add explicit clamping [0.05, 0.20]
- Void: Exact zero → Threshold (0.01)

---

**Status:** Ready to begin migration after checklist complete
