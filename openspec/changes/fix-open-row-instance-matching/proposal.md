## Why

The PureScript compiler produces false positive `NoInstanceFound` errors when using `else` instances with open row patterns (e.g., `{ | r }`) in class heads. When a closed row type (like a record literal) is matched against an open row pattern with a rigid variable tail, the instance matcher incorrectly returns `Apart` instead of `Match`, preventing valid instances from being found. This breaks common patterns in heterogeneous folding and row-based generic programming.

## What Changes

- Fix `match_row_type` in the constraint solver to treat rigid variables in instance patterns as matching any row type
- When the wanted row tail is a rigid variable (from `freshen_instance_signature`), return `Match` instead of `Apart`
- Two match arms in `match_row_type` are affected: `Additional => Open(wanted_tail)` and `Closed => Open(wanted_tail)`

## Capabilities

### New Capabilities
<!-- None - this is a bug fix, no new capabilities -->

### Modified Capabilities
<!-- None - this is a bug fix, no spec-level behavior changes -->

## Impact

- `compiler-core/checking/src/core/constraint/matching.rs` - `match_row_type` function (2 locations)
- Existing instance resolution tests may need snapshot updates if behavior changes are visible
- No API changes, no breaking changes
