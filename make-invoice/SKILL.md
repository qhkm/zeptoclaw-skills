---
name: make-invoice
version: 1.0.0
description: Generate professional HTML/PDF invoices from order data and email them to customers.
author: ZeptoClaw
license: MIT
tags:
  - invoice
  - billing
  - pdf
  - ecommerce
env_needed:
  - name: INVOICE_COMPANY_NAME
    description: Your company name (e.g. Kedai Ahmad Sdn Bhd)
    required: true
  - name: INVOICE_COMPANY_ADDRESS
    description: Company address for invoice header
    required: false
  - name: INVOICE_CURRENCY
    description: "Currency code: MYR, SGD, USD, THB, IDR"
    required: false
metadata: {"zeptoclaw":{"emoji":"🧾","requires":{"anyBins":["python3","wkhtmltopdf"]}}}
---

# Make Invoice Skill

Generate professional invoices as HTML or PDF. Works for any order — Lazada, Shopee, manual, or custom.

## Setup

```bash
export INVOICE_COMPANY_NAME="Kedai Ahmad Sdn Bhd"
export INVOICE_COMPANY_ADDRESS="No 12, Jalan Usahawan, 47500 Subang Jaya, Selangor"
export INVOICE_CURRENCY="MYR"
```

Install wkhtmltopdf (for PDF output):
```bash
# macOS
brew install wkhtmltopdf

# Ubuntu/Debian
sudo apt install wkhtmltopdf
```

## Generate HTML Invoice

```bash
cat > /tmp/invoice.py << 'EOF'
import os, sys, json
from datetime import datetime

data = json.load(sys.stdin)
company = os.environ.get("INVOICE_COMPANY_NAME", "My Company")
address = os.environ.get("INVOICE_COMPANY_ADDRESS", "")
currency = os.environ.get("INVOICE_CURRENCY", "MYR")
inv_no = data.get("invoice_number", f"INV-{datetime.now().strftime('%Y%m%d%H%M')}")
customer = data.get("customer", {})
items = data.get("items", [])
total = sum(i["quantity"] * i["unit_price"] for i in items)

rows = "\n".join(f"""
  <tr>
    <td>{i['description']}</td>
    <td style="text-align:right">{i['quantity']}</td>
    <td style="text-align:right">{currency} {i['unit_price']:.2f}</td>
    <td style="text-align:right">{currency} {i['quantity']*i['unit_price']:.2f}</td>
  </tr>""" for i in items)

print(f"""<!DOCTYPE html>
<html><head><meta charset="utf-8"><style>
  body{{font-family:Arial,sans-serif;max-width:800px;margin:40px auto;color:#333}}
  h1{{color:#1a1a1a}} table{{width:100%;border-collapse:collapse;margin-top:20px}}
  th{{background:#f5f5f5;padding:10px;text-align:left;border-bottom:2px solid #ddd}}
  td{{padding:8px 10px;border-bottom:1px solid #eee}}
  .total{{font-weight:bold;font-size:1.1em}} .meta{{color:#666;font-size:0.9em}}
</style></head><body>
<h1>INVOICE</h1>
<p><strong>{company}</strong><br>{address}</p>
<hr>
<table><tr><td>
  <p class="meta">Bill To:</p>
  <strong>{customer.get('name','')}</strong><br>
  {customer.get('email','')}<br>{customer.get('address','')}
</td><td style="text-align:right">
  <p class="meta">Invoice No: <strong>{inv_no}</strong></p>
  <p class="meta">Date: {datetime.now().strftime('%d %B %Y')}</p>
  <p class="meta">Due: {data.get('due_date', 'Upon Receipt')}</p>
</td></tr></table>
<table><tr>
  <th>Description</th><th style="text-align:right">Qty</th>
  <th style="text-align:right">Unit Price</th><th style="text-align:right">Amount</th>
</tr>{rows}
<tr><td colspan="3" style="text-align:right" class="total">TOTAL</td>
    <td style="text-align:right" class="total">{currency} {total:.2f}</td></tr>
</table>
<p style="margin-top:40px;color:#666">{data.get('notes','Thank you for your business!')}</p>
</body></html>""")
EOF

# Example: generate invoice from JSON
cat << 'INVOICE_DATA' | python3 /tmp/invoice.py > /tmp/invoice-INV001.html
{
  "invoice_number": "INV-2026-001",
  "customer": {
    "name": "Siti Rahimah binti Aziz",
    "email": "siti@example.com",
    "address": "Kuala Lumpur, Malaysia"
  },
  "items": [
    {"description": "Premium Widget x2", "quantity": 2, "unit_price": 49.90},
    {"description": "Shipping", "quantity": 1, "unit_price": 8.00}
  ],
  "notes": "Payment via bank transfer to Maybank 5612-1234-5678"
}
INVOICE_DATA

echo "Invoice saved: /tmp/invoice-INV001.html"
```

## Convert HTML to PDF

```bash
wkhtmltopdf --quiet /tmp/invoice-INV001.html /tmp/invoice-INV001.pdf
echo "PDF saved: /tmp/invoice-INV001.pdf"
```

## Email the Invoice (with send-email skill)

```bash
ATTACHMENT=$(base64 -i /tmp/invoice-INV001.pdf)

curl -s -X POST "https://api.resend.com/emails" \
  -H "Authorization: Bearer $RESEND_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{
    \"from\": \"$EMAIL_FROM\",
    \"to\": [\"siti@example.com\"],
    \"subject\": \"Invoice INV-2026-001 from $INVOICE_COMPANY_NAME\",
    \"text\": \"Dear Siti,\n\nPlease find your invoice attached.\n\nTotal: MYR 107.80\n\nThank you!\",
    \"attachments\": [{
      \"filename\": \"invoice-INV001.pdf\",
      \"content\": \"$ATTACHMENT\"
    }]
  }" | jq -r '.id'
```

## Batch Invoices from CSV

```bash
# invoices.csv: invoice_number,customer_name,customer_email,description,quantity,unit_price
while IFS=, read -r inv_no name email desc qty price; do
  cat << EOF | python3 /tmp/invoice.py > "/tmp/invoice-${inv_no}.html"
{"invoice_number":"$inv_no","customer":{"name":"$name","email":"$email"},"items":[{"description":"$desc","quantity":$qty,"unit_price":$price}]}
EOF
  wkhtmltopdf --quiet "/tmp/invoice-${inv_no}.html" "/tmp/invoice-${inv_no}.pdf"
  echo "Generated: invoice-${inv_no}.pdf"
done < invoices.csv
```

## Tips

- Use `invoice_number` as a unique ID — include year prefix (e.g. `INV-2026-001`)
- Store generated invoice paths in your database for retrieval
- For SST (Malaysian sales tax), add a tax row: `{"description": "SST 8%", "quantity": 1, "unit_price": total * 0.08}`
- wkhtmltopdf renders exactly like Chrome — test HTML in browser first
- Keep the Python script as a file (`~/.zeptoclaw/scripts/invoice.py`) for reuse
