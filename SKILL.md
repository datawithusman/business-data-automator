---
name: business-data-automator
version: 1.0.0
description: |
  Convert messy business spreadsheets into clean databases, dashboards, and automated
  workflows. Built for developers and consultants working with non-tech clients who
  run their entire operation on Excel. Covers data profiling, cleaning patterns,
  schema design, dashboard architecture, and common automation recipes for small
  businesses: fleet operations, retail, clinics, delivery, construction.
license: MIT
compatibility: claude-code opencode cursor windsurf
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Grep
  - Glob
---

# Business Data Automator

You help developers turn messy client spreadsheets into real systems. The user is usually a freelancer, consultant, or small agency working with a non-tech business owner who runs everything on Excel and manual processes.

## How to think about this work

The client does not care about pandas, normalization, or architecture. They care about hours saved and numbers they can check at 9 AM on Monday. Frame every decision in those terms.

A good deliverable has three layers:

1. **Clean data** that matches how the business actually thinks about itself
2. **A dashboard** the owner can open on their phone without training
3. **One automation** that removes the most boring task they do every week

Do not build all three at once. Start with the data, get paid, then upsell the dashboard and automation.

## Step 1: Data profiling

Before writing any code, ask the client to send the last 3 months of Excel files. Not a sample. The real files. Look for:

- **Multiple sheets** named "final", "final 2", "FINAL real" (this tells you their real workflow is messy versioning)
- **Free text columns** that should be categories (rider names spelled 5 ways, supplier names with typos)
- **Date columns** stored as text, sometimes in `MM/DD/YYYY`, sometimes in `DD-MM-YYYY`
- **Currency** mixed with text like "SAR 1,200" or "1200 riyal" in the same column
- **Merged cells** used for visual grouping instead of a real category column
- **Empty rows** between sections that humans use as visual breaks
- **Calculated columns** that reference other columns which sometimes got deleted

Ask the client: "Walk me through how you fill this in on Monday morning." Record it. Every assumption you make about their data will be wrong unless you watch them work.

## Step 2: Cleaning patterns

Use these patterns in pandas. They handle 90% of real-world messy Excel files.

### Date columns with mixed formats

```python
def clean_dates(df, column):
    """Handle mixed date formats in one column."""
    parsed = pd.to_datetime(df[column], format='mixed', errors='coerce', dayfirst=True)
    # Anything that failed parsing is probably text like "March 15"
    mask = parsed.isna() & df[column].notna()
    if mask.any():
        parsed[mask] = pd.to_datetime(df.loc[mask, column], errors='coerce')
    return parsed
```

### Free text categories

```python
def standardize_categories(df, column, mapping):
    """Map typos and variants to one canonical value."""
    df[column] = (
        df[column]
        .str.strip()
        .str.lower()
        .map(mapping)
        .fillna(df[column])  # keep original if no mapping
    )
    return df
```

Build the mapping by listing unique values first. Show the client the list and ask "which of these are the same thing?" They will tell you in 30 seconds. Do not guess.

### Currency stored as text

```python
def clean_currency(df, column):
    """Convert 'SAR 1,200' and '1200 riyal' to float."""
    return (
        df[column]
        .astype(str)
        .str.replace(r'[^\d.]', '', regex=True)
        .replace('', '0')
        .astype(float)
    )
```

### Merged cells and empty rows

```python
def drop_blanks_and_fill(df):
    """Drop fully empty rows, then forward-fill merged-cell groupings."""
    df = df.dropna(how='all')
    # Forward fill category columns that appeared merged
    category_cols = [c for c in df.columns if 'category' in c.lower() or 'type' in c.lower()]
    df[category_cols] = df[category_cols].ffill()
    return df
```

## Step 3: Schema design

Non-tech businesses think in nouns they use in conversation. Listen to how the client talks: "we had 14 deliveries yesterday for Al-Marai, three drivers were late." That sentence gives you 4 tables:

- `deliveries`
- `clients` (Al-Marai is one)
- `drivers`
- `shifts` (because "late" only makes sense against a scheduled time)

Resist normalization for its own sake. A single `deliveries` table with client_name and driver_name as columns is fine if the business has under 5,000 records per month. You can normalize later if they actually grow.

A common mistake: building a perfect 12-table schema the client cannot understand. Show them the tables on paper. If they cannot explain what each one is for, you over-engineered.

## Step 4: Dashboard architecture

For small businesses, the dashboard is the product. They will not open a database. They will open a dashboard.

Two solid options:

**Option A: Metabase or Superset on a VPS**
- Best when: client has a server, wants full ownership, no recurring SaaS cost
- Stack: Docker + Postgres + Metabase on a $10/month VPS
- Setup time: 2 to 4 hours
- Maintenance: low, updates every 6 months

**Option B: Streamlit or Dash web app**
- Best when: client wants custom branding, mobile-friendly, or wants it embedded in their existing site
- Stack: Streamlit + Postgres + Cloudflare Tunnel on a VPS
- Setup time: 1 to 3 days
- Maintenance: medium

**Option C: Next.js + Tremor + Postgres**
- Best when: client wants a polished product they can resell, or you are building a vertical SaaS from one client's playbook
- Stack: Next.js 14 + Tremor + Postgres + Vercel
- Setup time: 1 to 3 weeks
- Maintenance: medium

Pick based on what the client pays for and what you can support long-term. Do not pick Next.js if you cannot commit to updates.

### Dashboard rules

Every dashboard must have:

1. **One headline number** at the top (deliveries today, revenue this week, fleet utilization)
2. **A trend** next to the headline (vs yesterday, vs last week)
3. **Three to five charts** maximum. More than that and the owner stops looking.
4. **A table view** for drilling down when they want details
5. **Mobile layout** that works. Owners check dashboards on their phones at 11 PM.

What kills dashboards:
- Too many charts
- No mobile layout
- Numbers the client cannot tie back to their gut feel (then they stop trusting it)
- Stale data with no timestamp showing when it last updated

## Step 5: Automation recipes

These are the automations that actually get used in small businesses. Pick one based on what the client complains about most.

### Daily KPI email

Every morning at 7 AM, send a one-line email with yesterday's numbers. Owners love this.

```python
import smtplib
from email.mime.text import MIMEText
import schedule
import time

def send_daily_kpi():
    yesterday = pd.read_sql("""
        SELECT
            COUNT(*) AS deliveries,
            SUM(CASE WHEN status = 'late' THEN 1 ELSE 0 END) AS late_count,
            AVG(delivery_time_min) AS avg_time
        FROM deliveries
        WHERE DATE(created_at) = CURRENT_DATE - 1
    """, engine).iloc[0]

    body = f"""Yesterday:
    Deliveries: {yesterday.deliveries}
    Late: {yesterday.late_count}
    Avg time: {yesterday.avg_time:.0f} min"""

    msg = MIMEText(body)
    msg['Subject'] = f"Daily KPI - {yesterday.deliveries} deliveries"
    msg['From'] = "ops@client.com"
    msg['To'] = "owner@client.com"

    with smtplib.SMTP('smtp.client.com', 587) as s:
        s.starttls()
        s.login(USERNAME, PASSWORD)
        s.send_message(msg)

schedule.every().day.at("07:00").do(send_daily_kpi)
while True:
    schedule.run_pending()
    time.sleep(60)
```

### Late delivery alert

When a delivery crosses a threshold, send a WhatsApp or SMS. Use Twilio or the WhatsApp Business API.

```python
def check_late_deliveries():
    late = pd.read_sql("""
        SELECT driver_name, client_name, scheduled_time
        FROM deliveries
        WHERE status = 'in_progress'
          AND NOW() - scheduled_time > INTERVAL '30 minutes'
    """, engine)

    for _, row in late.iterrows():
        send_whatsapp(
            to=OWNER_PHONE,
            message=f"{row.driver_name} is 30+ min late for {row.client_name}"
        )
```

### Weekly Excel-to-database sync

Most clients will not stop using Excel cold turkey. Build a sync script that runs weekly, pulls their Excel from Google Drive or Dropbox, and updates the database.

```python
import pandas as pd
from sqlalchemy import create_engine

def sync_excel_to_db():
    excel_path = download_from_drive("Weekly_Operations.xlsx")
    df = pd.read_excel(excel_path, sheet_name="Deliveries")
    df = clean_dates(df, 'date')
    df = standardize_categories(df, 'status', STATUS_MAPPING)
    df = clean_currency(df, 'amount')

    engine = create_engine(DATABASE_URL)
    df.to_sql('deliveries', engine, if_exists='append', index=False)
    log.info(f"Synced {len(df)} rows")
```

## Common client types and what they actually need

### Fleet and delivery (Talabat-style)
- They want: rider KPIs, late delivery tracking, weekly supplier ratings
- Build: deliveries table, riders table, daily KPI dashboard, late alert
- Skip: ML predictions, route optimization (too complex for v1, pitch in v2)

### Retail and wholesale
- They want: stock levels, fast/slow movers, supplier performance
- Build: products, sales, inventory tables, reorder alerts
- Skip: full ERP, accounting integration

### Clinics and service businesses
- They want: appointments by day, no-show tracking, revenue per service
- Build: appointments, patients, services tables, daily schedule view
- Skip: full EHR, insurance claim handling

### Construction contractors
- They want: vendor payments, project expenses, milestone tracking
- Build: projects, vendors, payments tables, cash flow dashboard
- Skip: full project management suite

## Pricing your work

This skill does not set your prices, but here is what works in practice for small business automation work in 2026:

- **Data cleanup only**: $500 to $1,500
- **Database + dashboard**: $2,000 to $6,000
- **Database + dashboard + 1 automation**: $4,000 to $10,000
- **Monthly retainer (updates, fixes, new reports)**: $300 to $800/month

Charge the lower end if you have no portfolio in this niche yet. Raise prices after 2 to 3 completed projects.

## Anti-patterns to avoid

- Building a web app before understanding the data
- Pitching AI or ML before the basics are working
- Using Mongo or NoSQL when Postgres handles everything fine
- Skipping the dashboard and dumping data into a database the client never opens
- Over-documenting with 50-page specs. One page with the data flow diagram is enough.
- Hosting everything on AWS when a $10 VPS handles the load
- Using microservices for a 50-user internal tool

## What to ask the client in the first call

1. Send me your last 3 months of Excel files
2. Show me how you fill them in on a normal Monday
3. What is the one task you hate doing every week?
4. What number do you check first when you wake up?
5. Who else needs to see this data? (partners, investors, managers)
6. What happens when this spreadsheet breaks?

The answers tell you exactly what to build and in what order.

## Deliverables checklist

Before you hand off the project:

- [ ] Clean database with seed data for testing
- [ ] Dashboard the owner can open on their phone
- [ ] At least one automation that saves real time
- [ ] A simple README the client can read
- [ ] Backup script running daily
- [ ] One training session recorded on video
- [ ] 30 days of free fixes included in the price

Do not skip the training video. Clients forget everything from live sessions and the video saves you 10 support calls.