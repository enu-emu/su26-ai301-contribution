# Contribution [3]: [Fix Houses to use a 3D instead of 2D Region #29]

**Contribution Number:** 3
**Student:** Ena Salazar
**Issue:** [ModernUO #29 — Fix Houses to use a 3D instead of 2D Region](https://github.com/modernuo/ModernUO/issues/29)
**Status:** Phase I Complete — Issue understood and scoped

---

## Why I Chose This Issue

I chose this issue because it is open, effectively unclaimed (no comments, no linked pull request), and carries the labels `good first PR`, `help wanted`, and `bug`. The scope is clear and self-contained: the housing system builds its regions as if they were 2D (spanning the entire vertical column) instead of true 3D regions bounded to the house, which the issue notes allows exploitation and blocks features like townhouses. It's also a chance to work in a real C# game-server codebase (ModernUO is an Ultima Online server emulator) rather than a documentation change, so it stretches me a little further than my earlier contributions while still being narrowly scoped.

I also confirmed this issue has **no linked pull request** — the only thing referencing it is a separate issue (#268, "Octree for Regions"), not a PR — so it is genuinely available to work on.

---

## Understanding the Issue

### Problem Description

In ModernUO, every house has a `HouseRegion` — the spatial zone that controls access, banning, lockdowns, and other in-house rules. The issue asks for that region to be a true **3D** region instead of an effectively **2D** one. Today, a house's region covers its 2D footprint but extends across the *full* vertical range of the map, so the house "owns" all the space directly above and below it. This causes region conflicts and makes it impossible to cleanly stack structures (e.g. townhouses) or place other systems in the same footprint at a different height. The issue notes this can be exploited.

### Expected Behavior

A house's region should be bounded in the Z (vertical) axis to the actual house — its floor to its roof height — so that space clearly above or below the house is **not** part of the house region. This would let townhouses and other stacked/co-located systems integrate without region overlap and without the exploits the current behavior allows.

### Current Behavior

The house region is stored as `Rectangle3D[]`, but it is constructed by taking the house's 2D footprint and expanding it to span the entire map height. Concretely, in `HouseRegion.GetArea(...)` each 2D rectangle is passed through `Region.ConvertTo3D(...)`, which builds a `Rectangle3D` from `MinZ` to `MaxZ`:

```csharp
// Projects/Server/Regions/Region.cs
public const int MinZ = sbyte.MinValue;      // -128
public const int MaxZ = sbyte.MaxValue + 1;  //  128

public static Rectangle3D ConvertTo3D(Rectangle2D rect) =>
    new(new Point3D(rect.Start, MinZ), new Point3D(rect.End, MaxZ));
```

So although the type is 3D, the region behaves as 2D: it stretches from Z = -128 to Z = 128, the whole vertical column.

There's even a telling clue in the source — the house's Z is read and then commented out:

```csharp
// Projects/UOContent/Regions/HouseRegion.cs
private static Rectangle3D[] GetArea(BaseHouse house)
{
    var x = house.X;
    var y = house.Y;
    // int z = house.Z;   // <-- house height is deliberately dropped

    var houseArea = house.Area;
    var area = new Rectangle3D[houseArea.Length];

    for (var i = 0; i < area.Length; i++)
    {
        var rect = houseArea[i];
        area[i] = ConvertTo3D(new Rectangle2D(x + rect.Start.X, y + rect.Start.Y, rect.Width, rect.Height));
    }

    return area;
}
```

The commented `// int z = house.Z;` suggests the vertical bound was intended but never wired in.

### Affected Components

- **`Projects/UOContent/Regions/HouseRegion.cs`** — `GetArea(BaseHouse house)` is where the region's area is built. This is the primary place the fix will live: it needs to produce Z-bounded `Rectangle3D`s using the house's Z and height instead of calling the full-column `ConvertTo3D`.
- **`Projects/Server/Regions/Region.cs`** — defines `MinZ`/`MaxZ` and the `ConvertTo3D` helpers that flatten a 2D rectangle across the whole vertical range. Understanding this explains *why* the region is effectively 2D today.
- **`BaseHouse`** (`Projects/UOContent/Multis/BaseHouse.cs`) — source of `house.X`, `house.Y`, `house.Z`, and `house.Area`; likely also where a height/roof value would come from to bound the region.
- **Region consumers of `HouseRegion`** — many systems check `IsPartOf<HouseRegion>()` (spells like `Teleport`, `ShadowJump`, `NatureFury`; boats; house placement). These are worth noting because changing the region's vertical bounds could change what counts as "inside a house," so they're the surface area to watch during testing.
- **`dev-docs/regions.md`** — the project's own documentation of how regions and priorities work; useful reference for making the change consistent with the region model.

---

## Reproduction Process

_Planned for Week 2 (Phase II). Not started yet._

Initial plan:
- Fork `modernuo/ModernUO`, clone locally, add the upstream remote, and create a working branch.
- Confirm the build/run requirements for ModernUO (it's a .NET server; needs a client/data to fully run) and decide whether a full run is feasible on my hardware or whether I verify at the code level, as I did for CTSM.
- Reproduce the 2D-region behavior — either in a running server (stand on a spot far above/below a house and confirm you're still treated as "in" the house region) or by tracing the code path from `GetArea` → `ConvertTo3D` → the `MinZ..MaxZ` span.

---

## Solution Approach

_Detailed design planned for Week 2 (Phase II). Not started yet._

Early direction (to be validated):
- Replace the `ConvertTo3D` (full-column) call in `HouseRegion.GetArea` with a Z-bounded `Rectangle3D` built from the house's Z (floor) and an appropriate height (roof/ceiling), re-using the `// int z = house.Z;` value that's currently commented out.
- Confirm how ModernUO determines a house's vertical extent (fixed height vs. per-house/roof data) before hard-coding anything.
- Keep the change minimal and consistent with the region model documented in `dev-docs/regions.md`.

---

## Testing Strategy

_Planned for Week 2–3. Not started yet._

- Verify a point clearly above or below a house is no longer reported as part of the `HouseRegion`, while points inside still are.
- Sanity-check the `IsPartOf<HouseRegion>()` consumers (spell travel restrictions, house placement, boats) to ensure the tighter region doesn't break legitimate in-house behavior.
- Run ModernUO's existing test suite / CI where applicable.

---

## Implementation Notes

### Week 1 Progress (Phase I — Understand & Choose)

- Selected ModernUO issue #29 and confirmed it is open, has the `good first PR` / `help wanted` / `bug` labels, has no comments, and — importantly — has **no linked pull request** (only a cross-reference from issue #268, which is itself an issue, not a PR).
- Read the relevant source to understand the root cause: `HouseRegion.GetArea` builds the region by calling `Region.ConvertTo3D`, which spans `MinZ (-128)` to `MaxZ (128)`, making the region effectively 2D. Found the commented-out `// int z = house.Z;` line showing the vertical bound was intended but dropped.
- Documented the problem, expected vs. current behavior, and the affected components (above).

### Week 2 Progress (Phase II — Reproduce & Plan)

_Not started yet._

### Week 3 Progress (Phase III — Build & PR)

_Not started yet._

### Week 4 Progress (Phase IV — Review & Iterate)

_Not started yet._

---

## Pull Request

_Not opened yet — planned for Phase III._

---

## Learnings & Reflections

_To be completed as the contribution progresses. Week 1 takeaway: the label "good first PR" doesn't always mean the change is tiny — this one is well-scoped but touches a core spatial system, so the real Phase I work was reading enough of the region code to understand exactly why a `Rectangle3D[]` region can still behave as 2D._

---

## Resources Used

- [ModernUO issue #29](https://github.com/modernuo/ModernUO/issues/29)
- [ModernUO `HouseRegion.cs`](https://github.com/modernuo/ModernUO/blob/main/Projects/UOContent/Regions/HouseRegion.cs)
- [ModernUO `Region.cs` (`ConvertTo3D`, `MinZ`/`MaxZ`)](https://github.com/modernuo/ModernUO/blob/main/Projects/Server/Regions/Region.cs)
- [ModernUO `dev-docs/regions.md`](https://github.com/modernuo/ModernUO/blob/main/dev-docs/regions.md)
- [ModernUO CONTRIBUTING guide](https://github.com/modernuo/ModernUO/blob/main/CONTRIBUTING.md)
