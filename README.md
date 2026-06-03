# CanvasApps

All Canvas Apps related.

## Power Apps Canvas HTML template

Use this HTML with a **HTML text** control in Microsoft Power Apps Canvas apps.

```html
<div style="font-family:Segoe UI,Arial,sans-serif;padding:16px;border:1px solid #e1e1e1;border-radius:8px;background:#ffffff;">
  <h2 style="margin:0 0 8px 0;color:#323130;">Status Summary</h2>
  <p style="margin:0;color:#605e5c;">Hello, <strong>{{UserName}}</strong>!</p>
  <p style="margin:8px 0 0 0;color:#605e5c;">Your request <strong>{{RequestId}}</strong> is currently <span style="color:#107c10;"><strong>{{Status}}</strong></span>.</p>
</div>
```

### Example Power Apps formula

Set the HTML text control `HtmlText` property to:

```powerfx
Substitute(
    Substitute(
        Substitute(
            "<div style='font-family:Segoe UI,Arial,sans-serif;padding:16px;border:1px solid #e1e1e1;border-radius:8px;background:#ffffff;'><h2 style='margin:0 0 8px 0;color:#323130;'>Status Summary</h2><p style='margin:0;color:#605e5c;'>Hello, <strong>{{UserName}}</strong>!</p><p style='margin:8px 0 0 0;color:#605e5c;'>Your request <strong>{{RequestId}}</strong> is currently <span style='color:#107c10;'><strong>{{Status}}</strong></span>.</p></div>",
            "{{UserName}}",
            User().FullName
        ),
        "{{RequestId}}",
        Text(varRequestId)
    ),
    "{{Status}}",
    varStatus
)
```
