# MCP Server Migration Guide: Revamped UNITARES Framework

**Created:** November 16, 2025  
**Last Updated:** November 16, 2025  
**Status:** Migration Guide

## Overview

This guide provides step-by-step instructions for migrating the Governance MCP server from the current implementation to the revamped UNITARES framework with improved mathematics.

## Migration Summary

### Key Changes

1. **Void Integral (V)**: Pure integration → Leaky integrator (γ = 0.85)
2. **Integrity (I)**: Integrated computation → Direct computation (I = 1 - S)
3. **Energy (E)**: Differential equation → Direct normalized (optional)
4. **Lambda Control**: Add explicit clamping
5. **Void Detection**: Exact zero → Threshold-based

### Benefits

- ✅ Fixes accumulation weakness
- ✅ Prevents unbounded growth
- ✅ Guarantees bounded state variables
- ✅ Enables system recovery
- ✅ Simplifies implementation

---

## Pre-Migration Checklist

- [ ] Backup current MCP server code
- [ ] Backup existing CSV files
- [ ] Review current implementation
- [ ] Understand revamped framework
- [ ] Plan testing strategy
- [ ] Prepare rollback plan

---

See full guide for complete migration steps, code examples, and testing procedures.
