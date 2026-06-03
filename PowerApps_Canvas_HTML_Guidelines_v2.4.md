# Power Apps Canvas HTML Development Guidelines


## User Profile & Preferences


### Developer Background
- **Role**: Microsoft Power Platform Developer
- **Specialization**: Power Apps, Power Automate
- **Learning Focus**: AI integration
- **Location**: Vaughan, Ontario, CA


### Standard Variable Naming Conventions

#### Prefix Meanings

| Prefix | Type | Scope | Set With |
|---|---|---|---|
| `nf` | **Named Formula** | App-wide, always calculated | `App.Formulas` |
| `v` | **Context Variable** | Screen or component scope | `Set()` / `UpdateContext()` |
| `col` | **Collection** | App-wide table/list | `Collect()` / `ClearCollect()` |
| `drp_` | **Dropdown control** | Control naming convention | n/a |

#### Common Named Formula Variables
- `nfPrimaryColorText` — Primary brand color (CSS string for HTML)
- `nfAccentColorText` — Accent/secondary color (CSS string for HTML)
- `nfThridColorText` — Third brand color (CSS string for HTML) — note: "Thrid" is intentional
- `nfLGreyText` — Light grey (CSS string for HTML)
- `nfPrimaryColor` — Primary brand color (RGBA for Power Fx)
- `nfAccentColor` — Accent color (RGBA for Power Fx)
- `nfThirdColor` — Third color (RGBA for Power Fx)
- `nfLGrey`, `nfGrey`, `nfDGrey` — Light / medium / dark grey (RGBA for Power Fx)
- `nfAppName` — Application name
- `nfAppVersion` — Application version number
- `nfHtmlReset` — HTML body margin reset string

#### Common Context Variables
- `vSelected[Entity]` — Selected item (e.g. `vSelectedOrder`, `vSelectedFormOrder`)
- `v[VariableName]` — General boolean/state variables (e.g. `vShowSidePaneLeft`)

#### Collections
- `col[EntityName]` — Collection variables (e.g. `colNotes`, `colOrders`, `colAudit`)

#### Control Naming
- `drp_[name]` — Dropdown controls (e.g. `drp_regionSelector`)


---


## HTML in Canvas Apps: Limitations & Best Practices


### Key Limitations
1. **No JavaScript execution** - `<script>` tags are completely ignored. Only static HTML and inline CSS supported.
2. **No `<style>` blocks (with one exception)** - The PA renderer strips `<style>` blocks when placed inside a `$" "` interpolated string because Power Apps intercepts `{ }` as Power Fx expressions — so CSS rules containing `{ }` (like `body { margin: 0; }`) are consumed before the HTML renders. **The only working workaround** is to prepend a plain (non-interpolated) string: `"<style>body{margin:0;padding:0;}</style>" & $"...your html..."`. All other CSS must be inline `style=''` on each element.
3. **No full document tags** - `<html>`, `<head>`, `<body>`, `<meta>`, `<title>` are all stripped. PA only renders the HTML fragment.
4. **No external resources** - Cannot load external CSS files, fonts, or images via URL.
5. **Limited interactivity** - Cannot handle click events or dynamic behaviors. Use native Power Apps controls for buttons and inputs.
6. **Text encoding** - Use `EncodeHTML()` for user-generated content to prevent HTML injection.
7. **Performance** - Complex HTML can impact app performance; keep markup lean.
8. **Quote rules** - Use single quotes `' '` for all HTML attributes outside `{ }`. Inside `{ }` Power Fx expressions, use normal double quotes `" "` — no escaping needed. Never use `\"` (backslash escape) or `""` (doubled quotes) inside `{ }` — neither pattern works in Power Apps.


### ⚠️ WebView Body Margin — Default Browser Spacing

> **Issue**: The HTML content in a Power Apps HtmlText control renders inside a WebView component that applies the browser's default User Agent (UA) stylesheet. This adds `margin: 8px` to the `<body>` element automatically ��� space you never asked for.

**Symptoms:**
- A visible gap appears at the bottom (and sometimes top) of the HTML control
- In gallery rows, the highlight/selection background only covers ~80% of the row height vertically
- Table rows appear to have extra padding below them with no obvious CSS cause

**Why `<style>body{margin:0;}` alone does not work inside `$" "`:**
When the string starts with `$"`, Power Apps treats everything inside `{ }` as a Power Fx expression. The CSS rule `body{margin:0;padding:0;}` contains curly braces — Power Apps intercepts them as empty expressions before the HTML ever renders, breaking the style block silently.

**The Fix — Split into two strings:**
```powerfx
"<style>body{margin:0;padding:0;}</style>" & $"<table style='...'>...</table>"
```
The first part is a plain string (no `$`) — Power Apps passes `{ }` through as literal CSS. The second part is `$" "` for Power Fx interpolation as normal.

**Add to Named Formula for reuse everywhere:**
```powerfx
// In App.Formulas:
nfHtmlReset = "<style>body{margin:0;padding:0;}</style>";

// Then every HTML control becomes:
nfHtmlReset & $"<table style='...'>...</table>"
```

**Apply to ALL HtmlText controls in the app** — header tables, gallery rows, cards, banners. This one reset eliminates the WebView margin issue universally.

| Approach | Reliability |
|---|---|
| `margin: 0 0 -8px 0` on `<table>` | ⚠️ Fragile — varies by device/WebView version |
| `height: 100%` on `<table>` | ⚠️ Partial — does not reset the body margin |
| Setting PaddingBottom = 0 on the control | ⚠️ Sometimes works, not always accessible |
| `"<style>body{margin:0;padding:0;}</style>" &` | ✅ Definitive — overrides UA stylesheet directly |


### Unsupported CSS Properties
The following CSS properties are silently stripped or broken in the PA HtmlText renderer:
- `position: fixed` — layout collapses or disappears
- `position: sticky` — not supported
- `backdrop-filter: blur()` — silently ignored
- `clip-path` — not rendered
- `inset: 0` shorthand — not supported; use `top/right/bottom/left` explicitly
- `@keyframes` / CSS animations / `animation` / `transition` — no animation support
- `min-height: 100vh` — expands to full browser viewport, causes scrollbar (see Height Rules below)
- `z-index` — ignored since positioning is unsupported
- `transform` — not supported
- `filter` — not supported
- `grid-template-columns` / CSS Grid — limited support; use `<table>` instead


### Supported CSS Properties ✅
The following are confirmed to work reliably:
- All inline `style=''` attributes
- `background-color`, `background` with simple `linear-gradient()` (2–3 stops)
- `border`, `border-radius`, `border-left/right/top/bottom`
- `box-shadow`
- `font-family`, `font-size`, `font-weight`, `color`, `letter-spacing`, `text-transform`, `line-height`
- `padding`, `margin`, `width`, `height` (fixed `px` or `%`)
- `display: flex`, `flex-direction: row/column`, `align-items`, `justify-content`, `flex: 1`, `flex-shrink`
- `gap` on flex containers — mostly works when `nfHtmlReset` is prepended
- `display: inline-block`
- `overflow: hidden`, `text-overflow: ellipsis`, `white-space: nowrap`
- `box-sizing: border-box`
- `opacity`
- `rgba()` colors
- `vertical-align`, `text-align`
- `position: absolute` — works inside `<td>` and `<div>` containers; causes collapse only on the outermost root wrapper element
- HTML entities (`&#8594;`, `&middot;`, `&nbsp;`, `&#8226;`)
- Emojis as icon substitutes


### When to Use HTML
- Custom table layouts with precise styling
- Rich text formatting not achievable with native controls
- Complex visual layouts (cards, banners, headers, landing pages)
- Dynamic content rendering with collections


### When NOT to Use HTML
- Interactive buttons or forms (use native Power Apps controls)
- Data entry fields
- Charts or visualizations (use native charting)
- Simple text displays


---


## Height & Scroll Rules


> **Issue: HtmlText control causes vertical scrollbar**


The Power Apps HtmlText control **auto-sizes its height to match the full rendered height of the HTML content**. There is no overflow clipping — if the content is taller than the control, a scrollbar appears.


**Causes:**
- `min-height: 100vh` — expands to the full browser viewport height, far taller than the PA control
- `padding: 48px` on outer wrapper — adds ~96px of extra vertical space
- `<br/>` line breaks inside headings — each adds a full line height
- Large `margin-bottom` values stacked across sections (e.g. 32px + 36px + 32px = 100px in gaps alone)
- `height: 100vh` without a matching control height


**Fix:**
- Remove all `min-height`
- Keep outer padding to `16px–24px` max
- Keep section `margin-bottom` to `8px–16px` max
- To fill the full control height in Power Fx, use: `height: {Parent.Height}px`
- Use `overflow: hidden` on the outer wrapper to prevent content bleed


---


## Table Structure Standards


### Table Header Format


```powerfx
"<style>body{margin:0;padding:0;}</style>" &
$"<table style='table-layout: fixed; width: 100%; border-collapse: collapse; border-spacing: 0; margin: 0; padding: 0; font-family: Segoe UI, -apple-system, sans-serif; border: 0; border-bottom: 3px solid {nfAccentColorText};'>"
  <colgroup>
    <!-- C1: Column Name --><col style='width: X%'>
    <!-- C2: Column Name --><col style='width: Y%'>
  </colgroup>
  <tr style='border: 0;'>
    <!-- H1 --><th style='background-color: {nfPrimaryColorText}; color: #ffffff; font-weight: 600; border: 0; border-bottom: 0; padding: 12px 8px; text-align: center; vertical-align: middle; font-size: 12px;'>HEADER TEXT</th>
    <!-- H2 --><th style='...'></th>
  </tr>
</table>"
```


**Key Points:**
- Always prepend `"<style>body{margin:0;padding:0;}</style>" &` before the `$" "` string
- Use `<colgroup>` with comments: `<!-- C1: Column Name -->`
- Header comments: `<!-- H1 -->`, `<!-- H2 -->`, etc.
- Background uses `{nfPrimaryColorText}` variable
- Fixed table layout with percentage widths
- Accent bottom border: `border-bottom: 3px solid {nfAccentColorText}`


### Table Row Format


```powerfx
"<style>body{margin:0;padding:0;}</style>" &
$"<table style='table-layout: fixed; width: 100%; border-collapse: collapse; margin: 0; padding: 0; font-family: Segoe UI, sans-serif; border-bottom: 1px solid #e2e8f0;'>
  <colgroup>
    <!-- Same colgroup as header -->
  </colgroup>
  <tr style='cursor: pointer; background: {If(ThisItem.IsSelected, "#e0f2fe", "#ffffff")};'>
    <!-- R1 -->
    <td style='padding: 10px 6px; vertical-align: middle; text-align: center; font-size: 12px; color: #64748b;' title='{ThisItem.FieldName}'>
      {ThisItem.FieldName}
    </td>
  </tr>
</table>"
```


**Key Points:**
- Always prepend `"<style>body{margin:0;padding:0;}</style>" &` — eliminates WebView body margin gap
- Row comments: `<!-- R1 -->`, `<!-- R2 -->`, etc. (sequential numbering)
- Background on `<tr>` not `<table>` — ensures full row height is highlighted on selection
- Use `title` attribute for tooltip on overflow content
- Use `{ThisItem.FieldName}` for gallery template context
- Only use `EncodeHTML()` for fields that may contain special characters


### Selected Row Highlighting


```powerfx
<tr style='cursor: pointer; background: {If(ThisItem.IsSelected, "#e0f2fe", "#ffffff")};'>
```


**Features:**
- Light blue background (`#e0f2fe`) when selected
- Background set on `<tr>` not `<table>` for correct full-height fill
- Optionally add colored left border: `border-left: {If(ThisItem.IsSelected, "4px solid " & nfPrimaryColorText, "4px solid transparent")}`


---


## Common Component Patterns


### 1. Page Header / Nav Bar


```powerfx
$"<div style='font-family: Segoe UI, -apple-system, system-ui, sans-serif; background: {nfPrimaryColorText}; padding: 0; margin: 0; height: 80px; width: 100%; box-shadow: 0 2px 8px rgba(0,0,0,0.08); box-sizing: border-box;'>
  <div style='padding: 24px 48px; display: flex; align-items: center; justify-content: space-between; height: 100%; box-sizing: border-box;'>
    <div style='display: flex; align-items: center; gap: 16px;'>
      <div style='width: 40px; height: 40px;'></div>
      <div>
        <div style='font-size: 22px; font-weight: 700; color: #ffffff; letter-spacing: -0.5px;'>{nfAppName}</div>
        <div style='font-size: 11px; color: rgba(255,255,255,0.65); font-weight: 600; background: rgba(255,255,255,0.1); padding: 2px 8px; border-radius: 4px; display: inline-block; margin-top: 4px;'>{nfAppVersion}</div>
      </div>
    </div>
  </div>
</div>"
```


### 2. Empty State Message


```powerfx
$"<div style='display:flex;flex-direction:column;align-items:center;justify-content:center;padding:60px 20px;text-align:center;'>
  <div style='font-size:48px;margin-bottom:16px;opacity:0.3;'>📝</div>
  <div style='font-size:16px;font-weight:600;color:#666;margin-bottom:8px;'>No Items Found</div>
  <div style='font-size:13px;color:#999;'>Descriptive message about the empty state.</div>
</div>"
```


### 3. Notes / Comments Section with Collection


```powerfx
$"<div style='padding:0;margin:0;'>
  {If(
    IsEmpty(colNotes),
    "<div style='text-align:center;padding:60px 20px;'>No Notes Found</div>",
    Concat(colNotes,
      $"<div style='background:#ffffff;border:1px solid #e8e8e8;border-radius:8px;padding:16px;margin:0 0 6px 0;'>
        <div style='font-size:13px;color:#555;'>{EncodeHTML(desc)}</div>
      </div>"
    )
  )}
</div>"
```


### 4. Status Badge


```powerfx
<span style='display: inline-block; padding: 3px 24px; border-radius: 10px; font-size: 11px; font-weight: 600;
  background: {Switch(ThisItem.Status.Value,
    "Draft",     "#f1f5f9",
    "Pending",   "#fef3c7",
    "Completed", "#d1fae5",
    "Rejected",  "#fee2e2",
    "#f1f5f9")};
  color: {Switch(ThisItem.Status.Value,
    "Draft",     "#64748b",
    "Pending",   "#d97706",
    "Completed", "#059669",
    "Rejected",  "#dc2626",
    "#64748b")};'>
  {ThisItem.Status.Value}
</span>
```


**Status Color Map:**

| Status | Background | Text Color |
|---|---|---|
| Draft | `#f1f5f9` | `#64748b` |
| Pending | `#fef3c7` | `#d97706` |
| Completed | `#d1fae5` | `#059669` |
| Rejected | `#fee2e2` | `#dc2626` |


---


### 5. Collection / List Rendering

> This is the standard pattern for rendering repeating read-only data from a Power Apps collection into HTML. Since HTML in Canvas Apps has no interactivity (no click events, no editable fields), this pattern is **display-only**.

**Structure**: `If(IsEmpty(collection), "<empty state html>", Concat(collection, $"<item html>"))`

```powerfx
nfHtmlReset &
$"<div style='padding:0;margin:0;'>
  {If(
    IsEmpty(colAudit),

    // Empty state — shown when collection has no records
    "<div style='display:flex;flex-direction:column;align-items:center;justify-content:center;padding:60px 20px;text-align:center;'>
      <div style='font-size:48px;margin-bottom:16px;opacity:0.3;'>&#128203;</div>
      <div style='font-size:16px;font-weight:600;color:#666;margin-bottom:8px;'>No Records Available</div>
      <div style='font-size:13px;color:#999;'>Actions will appear here once recorded.</div>
    </div>",

    // Data state — Concat renders one block per record
    Concat(
      SortByColumns(colAudit, "timestamp", SortOrder.Descending),
      $"<div style='background:#ffffff;border:1px solid #e2e8f0;border-left:4px solid {nfPrimaryColorText};border-radius:6px;padding:12px 16px;margin:0 0 6px 0;'>
        <div style='display:flex;align-items:center;gap:10px;'>
          <div style='width:28px;height:28px;border-radius:50%;background:{nfPrimaryColorText};display:flex;align-items:center;justify-content:center;flex-shrink:0;'>
            <span style='font-size:12px;font-weight:700;color:#ffffff;'>{Upper(Left(user,1))}</span>
          </div>
          <div style='flex:1;'>
            <div style='font-size:13px;font-weight:600;color:#0f172a;'>{EncodeHTML(user)}</div>
            <div style='font-size:11px;color:#94a3b8;'>{EncodeHTML(email)}</div>
          </div>
          <div style='font-size:11px;color:#94a3b8;white-space:nowrap;'>{Text(timestamp, ""mmm dd, yyyy hh:mm ss"")}</div>
        </div>
        <div style='margin-top:8px;border-top:1px solid #f1f5f9;padding-top:8px;'>
          <span style='display:inline-block;background:{nfAccentColorText};color:#ffffff;font-size:11px;font-weight:600;text-transform:uppercase;letter-spacing:0.4px;padding:3px 9px;border-radius:20px;'>
            {EncodeHTML(action)}
          </span>
        </div>
      </div>"
    )
  )}
</div>"
```

**Key Points:**
- Always use `EncodeHTML()` on any field that may contain user-generated or special characters
- Use `SortByColumns()` inside `Concat()` to control display order
- The empty state uses a `$"..."` for the outer wrapper but the inner empty-state string is a plain `"..."` (no Power Fx interpolation needed there)
- `Concat()` does not support interactive elements — for editable lists, use a native Gallery control


---

All colors come from Named Formula variables defined in `App.Formulas`. Never hardcode brand colors — always reference these variables.

### Named Formula Variables (Canonical Reference)

The values below are the standard defaults. The **variable names never change** across apps; the values may differ per app.

```powerfx
// App info
nfAppVersion = "1.0.14";
nfAppName    = "App Name Here";

// Brand colors — Power Fx RGBA format (use in Power Fx color parameters)
nfPrimaryColor      = RGBA(0, 137, 196, 1);
nfPrimaryColorTrans = RGBA(0, 137, 196, 0.1);
nfAccentColor       = RGBA(7, 103, 155, 1);
nfThirdColor        = RGBA(251, 187, 54, 1);

// Brand colors — CSS string format (use inside HTML inline style='' attributes)
nfPrimaryColorText = "rgba(0,137,196,1)";
nfAccentColorText  = "rgba(7,103,155,1)";
nfThridColorText   = "rgba(251,187,54,1)";  // Note: "Thrid" is intentional — do not rename

// Neutral grays — CSS string format (use inside HTML inline style='' attributes)
nfLGreyText = "rgba(245,245,245,1)";

// Neutral grays — Power Fx RGBA format
nfLGrey = RGBA(245, 245, 245, 1);
nfGrey  = RGBA(200, 200, 200, 1);
nfDGrey = RGBA(150, 150, 150, 1);

// HTML body margin reset — prepend to every HtmlText control
nfHtmlReset = "<style>body{margin:0;padding:0}</style>";
```

### When to Use Each Format

| Situation | Use |
|---|---|
| Power Fx color parameter (e.g. `Fill`, `Color` on a native control) | `nfPrimaryColor`, `nfAccentColor`, `nfThirdColor`, `nfLGrey`, `nfGrey`, `nfDGrey` |
| HTML inline `style=''` attribute value | `{nfPrimaryColorText}`, `{nfAccentColorText}`, `{nfThridColorText}`, `{nfLGreyText}` |
| Prepend to HtmlText control | `nfHtmlReset & $"..."` |

### Status / Semantic Colors

These are not brand colors — they are semantic status indicators used in status badges and RAG (Red/Amber/Green) states. Hardcode these as-is:

| Status | Text Color | Background Color |
|---|---|---|
| Rejected / Critical | `#dc2626` | `#fee2e2` |
| Pending / Warning | `#d97706` | `#fef3c7` |
| Approved / Success | `#059669` | `#d1fae5` |
| Info / In Progress | `#2563eb` | `#dbeafe` |
| Neutral / Draft | `#64748b` | `#f1f5f9` |


---


## Typography Standards


### Font Families
- **Primary**: `Segoe UI, -apple-system, system-ui, sans-serif`


### Font Sizes
- **Page Titles**: 22–28px
- **Section Headers**: 20px
- **Body Text**: 14–17px
- **Labels**: 13–14px
- **Table Headers**: 12px
- **Table Cells**: 12px
- **Metadata / Captions**: 11–12px


### Font Weights
- **Bold**: 700 (headings)
- **Semi-bold**: 600 (labels, table headers, intake number)
- **Medium**: 500 (body text)
- **Regular**: 400 (table cell default)


---


## Common Issues & Solutions


### Issue: Only plain text visible, no styling
**Cause**: Used `<style>` block incorrectly or double quotes inside HTML string
**Solution**: Move all CSS to inline `style=''` attributes; use single quotes only; wrap in `$" "`


### Issue: Bottom gap / partial row highlight in gallery
**Cause**: WebView UA stylesheet applies `margin: 8px` to `<body>` automatically. Also, setting `background` on `<table>` instead of `<tr>` only fills content height, not the full gallery row height.
**Solution**:
1. Prepend `"<style>body{margin:0;padding:0;}</style>" &` before your `$" "` string
2. Set `background` on `<tr>`, not `<table>`
```powerfx
// Correct pattern:
"<style>body{margin:0;padding:0;}</style>" &
$"<table style='...'>
  <tr style='background: {If(ThisItem.IsSelected, "#e0f2fe", "#ffffff")};'>
    ...
  </tr>
</table>"
```


### Issue: `<style>body{margin:0;}` not working inside `$" "`
**Cause**: Power Apps intercepts `{ }` as Power Fx expressions inside `$" "` strings. The CSS curly braces in `body{margin:0;}` are consumed before HTML renders.
**Solution**: Split into two strings — plain string for the style reset, `$" "` for dynamic content:
```powerfx
"<style>body{margin:0;padding:0;}</style>" & $"<table>...</table>"
```


### Issue: HTML too tall, vertical scrollbar appears
**Cause**: `min-height: 100vh`, large padding/margins, or stacked `<br/>` breaks
**Solution**: Remove `min-height`; reduce outer padding to 16–24px; reduce section margins to 8–16px


### Issue: Layout breaks / elements disappear
**Cause**: Used `position: fixed`, `clip-path`, or `transform`
**Solution**: Replace with table-based layout or normal document flow. Note: `position: absolute` works inside `<td>` and `<div>` containers but will cause collapse if applied to the outermost root element.


### Issue: Text overflowing cells
**Solution**: Add `overflow: hidden; text-overflow: ellipsis; white-space: nowrap;`


### Issue: Special characters breaking layout
**Solution**: Use `EncodeHTML(Text(value))` for user-generated content


---


## Quote Rules in $" " Interpolated Strings


> **We always wrap HTML in `$" "` — this simplifies the quote rules to just 2 scenarios.**


### The Core Rule
Inside `$" "`, anything inside `{ }` is treated as **pure Power Fx**. Never escape quotes inside `{ }`.


### Scenario 1 — HTML Attributes (outside `{ }`)
Always use **single quotes** `' '` for all HTML `style`, `class`, `title`, and other attributes.


### Scenario 2 — Power Fx Expressions (inside `{ }`)
Always use **normal double quotes** `" "` inside `{ }` for all string values:
```powerfx
{Text(timestamp, "mmm dd, yyyy hh:mm ss")}
{If(condition, "value1", "value2")}
```


### ❌ Never Do These Inside `{ }`
```powerfx
// ❌ Wrong — backslash escape
{If(condition, \"value1\", \"value2\")}

// ❌ Wrong — doubled quotes
{If(condition, ""value1"", ""value2"")}

// ✅ Correct
{If(condition, "value1", "value2")}
```


### Quick Reference

| Location | Quote Style | Example |
|---|---|---|
| HTML attributes (outside `{}`) | Single `' '` | `style='font-size:13px;'` |
| Power Fx strings (inside `{}`) | Double `" "` | `{If(done, "Yes", "No")}` |
| Text() format strings (inside `{}`) | Double `" "` | `{Text(Now(), "mmm dd, yyyy")}` |


---


## End of Guidelines


**Last Updated**: April 30, 2026
**Version**: 2.4
