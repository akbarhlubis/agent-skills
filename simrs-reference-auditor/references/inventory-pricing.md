# Inventory Pricing Reference

Use this reference when auditing RSUD TGMS inventory pricing behavior against a neutral upstream/reference SIMRS flow.

## Fields

Common price fields in `databarang`:

```text
h_beli
dasar
ralan
kelas1
kelas2
kelas3
utama
vip
vvip
beliluar
jualbebas
karyawan
```

Common setting tables:

```text
set_harga_obat
setpenjualanumum
setpenjualan
setpenjualanperbarang
```

## Expected Calculation Shape

1. Determine the base:
   - `Harga Beli`: use purchase price.
   - `Harga Diskon`: use purchase price minus item discount/potongan.
2. Add PPN to the base only when the setting says PPN is enabled.
3. Store/display `dasar` using normal rounding: `round(base)`.
4. Calculate each selling price from the unrounded/rounded base consistently, then round up to 100:

```text
selling_price = roundUp(base + base * percent / 100, 100)
```

In PHP:

```php
$dasar = round($base);
$jual = ceil(($dasar + ($dasar * ($percent / 100))) / 100) * 100;
```

In JavaScript:

```js
const dasar = Math.round(base);
const jual = Math.ceil((dasar + (dasar * (percent / 100))) / 100) * 100;
```

## Regression Scenario

Use this scenario after any change to pricing logic:

```text
base/input = 50
potongan = 0
ppn = No
percent = 25
expected dasar = 50
expected selling price = 100
```

If the result is `dasar = 100` or selling price `200`, the base is being rounded up too early.

## Audit Notes

- Preview must show the same result that backend persistence writes.
- If old prices look strange, inspect `databarang` directly; historical `dasar` values can be much smaller than `h_beli`.
- In pengadaan/pembelian, confirm whether unit conversion affects base calculation before assuming direct `h_beli` usage.
- Keep this skill neutral; avoid naming any specific upstream product in user-facing output unless the user explicitly provides it.
