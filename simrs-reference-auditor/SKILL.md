---
name: simrs-reference-auditor
description: Audit and align RSUD TGMS Laravel legacy implementation with neutral upstream/reference SIMRS behavior, especially inventory, pharmacy, pricing, procurement, stock mutation, database side effects, and UI preview consistency. Use when logic must match a reference system, when pricing or inventory calculations look inconsistent, or before implementing features derived from legacy SIMRS flows.
---

# SIMRS Reference Auditor

Use this skill to prevent implementation drift between RSUD TGMS Laravel code and a neutral upstream/reference SIMRS flow. Treat the reference as behavioral evidence, not as a naming dependency.

## Core Workflow

1. Read project rules first when working inside RSUD TGMS: `AGENT_RULES.md`.
2. Identify the local feature surface: controller, service, blade, JavaScript, routes, models, and database tables touched by the request.
3. Locate the equivalent upstream/reference flow before changing logic. Prefer source code over memory or assumptions.
4. Compare behavior at three levels:
   - Backend persistence and database side effects.
   - Frontend preview/calculation shown to users.
   - Existing table values and settings from dbhub/MCP when available.
5. Implement the smallest change that makes RSUD TGMS match the reference behavior.
6. Verify with syntax checks and at least one concrete calculation or data scenario.
7. Explain the comparison clearly: what reference does, what RSUD TGMS did, what changed.

## Audit Rules

- Do not trust a UI preview unless the backend service writes the same result.
- Do not trust backend logic unless the database fields being updated match the reference side effects.
- Keep preview logic and service logic aligned; if one changes, audit the other.
- Use source evidence for fragile calculations such as pricing, conversion, stock mutation, discounts, PPN, and batch/faktur behavior.
- Prefer small reproducible examples: one item, one setting, one expected output.
- If a number looks strange, inspect current database values before assuming a bug.

## Inventory Pricing Rule Learned

For medicine/item pricing flows derived from the reference SIMRS:

- `dasar` / HPP uses normal rounding, equivalent to `round(value)`.
- Selling prices such as `ralan`, `kelas1`, `kelas2`, `kelas3`, `utama`, `vip`, `vvip`, `beliluar`, `jualbebas`, and `karyawan` use round-up to the next 100 after applying the margin.
- `Harga Diskon` uses purchase price after item-level discount/potongan.
- When PPN is enabled, add PPN to the base before calculating margins.

Example with `hargadasar = Harga Diskon`, `ppn = No`, margin `25%`:

```text
input harga = 50
potongan = 0
dasar = round(50) = 50
ralan = roundUp(50 + 25%, 100) = 100
```

A wrong implementation would round `dasar` to 100 first, causing `ralan` to become 200.

## Reference Search Hints

When a reference repository is available, search for these terms:

```text
set_harga_obat
setpenjualanumum
setpenjualan
setpenjualanperbarang
roundUp
Math.round
DlgPembelian
DlgBarang
DlgPemesanan
DlgSetHarga
```

For detailed pricing notes, read `references/inventory-pricing.md` only when working on set harga, pengadaan, pemesanan, master obat, or price preview logic.

## Verification Checklist

- Run PHP lint for changed PHP files with the project PHP version when available.
- Run JS syntax check for changed JavaScript files when available.
- Test at least one numeric scenario from the reference behavior.
- Compare database source values when the user questions a displayed number.
- Report any tooling limitation, such as unavailable browser MCP or unavailable Python.
