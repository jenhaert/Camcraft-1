# #769 - Part alignment from two perpendicular edge picks

## What was decided

Two perpendicular picks - an edge (or two points) for +Y and one for
+X - now define the setup's part alignment outright, in full 3D:
`pick::align_from_axes` builds the frame with the primary axis kept
exactly, the secondary orthogonalized against it, and +Z landing on
the picks' cross (the right-hand rule). The panel that was #557's
in-plane turn applies it (renamed `AlignPartFromAxes`); with one axis
picked the twist about it is free, so the work plane still carries the
turn (`turn_in_plane` unchanged, kept for its single-pick callers).
The pivot is the parts' centre; the ORIGIN is deliberately out of
scope (the work offset is placed separately, exactly as before).

## Why

The user's pick round (a 43 mm edge running down -Z picked as +Y, a
31.5 mm edge running -X picked as +X) carries complete frame
information that `turn_in_plane` threw away: it can only spin about
the work plane's normal, so an out-of-plane pick was refused ("runs
(nearly) along the work plane's normal") and the user was sent to a
plane-first workaround. The user's law: the two directions define the
axes, and where they run decides where +Z goes - no workaround, the
core flow must accept it. Mathematically the two directions compose
as a plain basis construction (`quat_from_axes` + conjugate), the same
building blocks the 3-2-1 geometry flow already uses.

## Considered and dropped

- **Keeping the left-handed refusal** for in-plane pick pairs: with
  full 3D freedom both picks are always honourable literally; refusing
  would again throw away information the user gave. A left-handed pair
  now lands the part on its head - visible, undoable, and exactly what
  the picks said. Pinned deliberately in
  `a_left_handed_pair_is_honoured_literally_not_refused`.
- **A max-skew refusal** (picks not perpendicular enough): the
  secondary lands within the skew it came with, reported by the
  landing proof; orthogonalization is the standard answer, not a
  refusal. Only (anti)parallel picks within 1 degree refuse, by name.
- **Setting the origin in the same step**: explicitly deferred by the
  user ("definition des nullpunktes ist dabei erstmal egal") - the
  work offset flow stays the one place origins are placed.
- **A new panel**: same rows, same picks, same flip toggles - only the
  meaning grew. The words under Apply now say which case applies
  ("turns N about Z" in-plane, "aligns the part" full 3D).

## What it would cost to change later

Every later alignment flow (origin picking, mate-like snaps) composes
on `align_from_axes`' contract: primary exact, secondary
orthogonalized, Z from the cross, pivot the parts' centre. Changing
the composition order or the literal-honouring law means re-deriving
the pins in `tests/align_axes.rs` (9 tests) and the UI pin
`two_picks_with_an_upright_edge_align_the_part_in_full_3d`.
