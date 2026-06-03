# Copilot Instructions — CanvasApps

This repo is a **centralized HTML reference and guideline library** for Power Apps Canvas App development. All HTML is written as Power Fx strings for use inside `HtmlText` controls.

---

## Full Guidelines

See [PowerApps_Canvas_HTML_Guidelines_v2.4.md](../PowerApps_Canvas_HTML_Guidelines_v2.4.md) for the complete reference.

---

## Standard Named Formula Variables

The variable **names** below are fixed across all apps. The **values** vary per app. Always use these variables — never hardcode brand colors.

> **Rule:** Only use the `nf` variables listed below. Do **not** invent new `nf` variables, and do **not** borrow variables from files in the `examples/` folder — those may contain app-specific variables that are not part of the standard set.

```powerfx
// App info
nfAppVersion = "1.0.14";
nfAppName    = "As-Built Bulk Edit";

// Brand colors — Power Fx RGBA format (use in Power Fx color parameters only)
nfPrimaryColor      = RGBA(0, 137, 196, 1);
nfPrimaryColorTrans = RGBA(0, 137, 196, 0.1);
nfAccentColor       = RGBA(7, 103, 155, 1);
nfThirdColor        = RGBA(251, 187, 54, 1);

// Brand colors — CSS string format (use inside HTML inline style='' attributes)
nfPrimaryColorText = "rgba(0,137,196,1)";
nfAccentColorText  = "rgba(7,103,155,1)";
nfThridColorText   = "rgba(251,187,54,1)";  // "Thrid" is intentional — do not rename

// Neutral grays — CSS string format
nfLGreyText = "rgba(245,245,245,1)";

// Neutral grays — Power Fx RGBA format
nfLGrey = RGBA(245, 245, 245, 1);
nfGrey  = RGBA(200, 200, 200, 1);
nfDGrey = RGBA(150, 150, 150, 1);

// HTML body margin reset — prepend to every HtmlText control
nfHtmlReset = "<style>body{margin:0;padding:0}</style>";
```

---

## The 3 Golden Rules

### 1. Always prepend `nfHtmlReset`
Every `HtmlText` control must start with the body margin reset:
```powerfx
nfHtmlReset & $"<table style='...'>...</table>"
```
Without it, the browser's default `margin: 8px` on `<body>` causes gaps and partial gallery row highlights.

### 2. Single quotes in HTML, double quotes inside `{ }`
```powerfx
// ✅ Correct
$"<div style='font-size:13px;color:{If(done, "#059669", "#dc2626")};'>text</div>"

// ❌ Wrong — double quotes on HTML attributes break the string
$"<div style="font-size:13px;">text</div>"

// ❌ Wrong — escaped quotes inside {} don't work in Power Apps
$"<div style='color:{If(done, \"green\", \"red\")};'>text</div>"
```

> **Rule:** Never use double quotes in HTML text content either. For quotation marks in visible text, use single quotes instead.
> ```powerfx
> // ✅ Correct — single quotes in text content
> $"<p style='...'>Click 'Upload' to continue.</p>"
>
> // ❌ Wrong — double quotes in text content will break the Power Fx string
> $"<p style='...'>Click "Upload" to continue.</p>"
> ```

### 3. Use variables, never hardcode brand colors
```powerfx
// ✅ Correct
$"<div style='background:{nfPrimaryColorText};'>"

// ❌ Wrong
$"<div style='background:rgba(0,137,196,1);'>"
```

---

## Examples

The `examples/` folder contains Power Fx snippets. Files are split into two categories:

### Always reference these files
These are structural patterns and must be consulted whenever generating or editing HTML for lists, galleries, or logs:

| File | What It Shows |
|---|---|
| `examples/listHeader` | Fixed-layout table header row (35 columns) |
| `examples/listItem` | Gallery row template with selection highlighting and status badge |
| `examples/logHistory` | Audit log with `Concat()` collection rendering + empty state |

### Never reference these files
The files below were added only to illustrate how HTML works inside Canvas Apps. Do **not** use them as design templates, do **not** copy variables from them, and do **not** base generated output on their structure:

| File | Why it exists |
|---|---|
| `examples/form` | HTML technique reference only |
| `examples/kpiCard` | HTML technique reference only |
| `examples/landingPage` | HTML technique reference only |
| `examples/log` | HTML technique reference only |
| `examples/navButton` | HTML technique reference only |

---

## Variable Naming Convention

| Prefix | Type | Example |
|---|---|---|
| `nf` | Named Formula (`App.Formulas`) | `nfPrimaryColorText`, `nfHtmlReset` |
| `v` | Context variable (`Set` / `UpdateContext`) | `vSelectedOrder`, `vShowPanel` |
| `col` | Collection | `colAudit`, `colOrders` |
| `drp_` | Dropdown control | `drp_regionSelector` |
