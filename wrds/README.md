# WRDS — get an account, download data, use it well

*For FIMA Lab members and students in our courses. Last checked 2026-10-13.*

**WRDS** (Wharton Research Data Services) is the data platform most finance papers use.
One login gives you many databases: stock prices, financial statements, analyst forecasts,
bonds, options and more. NYCU pays for a subscription, so it is free for you.

If the paper you are replicating uses US market data, it very likely came from here.

| Database | What it holds | Typical use |
|---|---|---|
| **CRSP** | Daily and monthly prices, returns and volume of US stocks, from 1925 | Returns, portfolios, event studies |
| **Compustat** | Financial statements of listed firms | Accounting ratios, firm characteristics |
| **IBES** | Analyst forecasts | Earnings surprises |
| **OptionMetrics** | Option prices and implied volatility | Volatility, option strategies |

NYCU does not subscribe to everything. Inside WRDS, databases shown in **blue** are open to you;
those in **grey** are not.

---

## 1. Get an account (approval is not instant: apply now)

1. Register at <https://wrds-www.wharton.upenn.edu/register/> with your **NYCU email address**,
   then click the link in the confirmation email.
2. Fill in the NYCU Library's WRDS application form: <https://forms.gle/hnP94tJ4fFSZC5o39>
3. Wait for the approval email and follow its instructions.

If the form has moved or approval does not arrive, ask the **NYCU Library** (WRDS contact:
yulin@lib.nycu.edu.tw, as of 2026).

---

## 2. Download data from the website

Example: daily stock data from CRSP.

1. Log in at <https://wrds-www.wharton.upenn.edu/login/>.
2. Click **Get Data**. Pick a database, or use the concept groups on the right
   (for example, *Banks* lists every bank-related database).
3. Open **CRSP** → **Stock / Security Files** → **Daily Stock File**.
4. Fill in the four steps of the query form:

| Step | What you choose | Advice |
|---|---|---|
| 1. Date range | Start and end dates | Daily data is large. Ask only for the period you need |
| 2. Company codes | Which firms | Enter tickers or PERMNOs, upload a list, or search the entire database |
| 3. Variables | Which columns | Select only what your paper uses: e.g. date, PERMNO, price, return, volume |
| 4. Output | File format | **csv**, dates as `YYYY-MM-DD` |

5. Click **Submit Form**. A large query takes a few minutes.
6. When the status shows *success*, download the output file.

**Use PERMNO, not the ticker, to identify a stock.** Tickers change and are reused;
a PERMNO stays with the same security for its whole life.

Variable definitions: *Variable Descriptions*, *Manuals and Overviews* and the *Data Dictionary*
on each database's page.

---

## 3. Or query it from Python

For anything you will run more than once, a script beats the website: it records exactly what you asked for.

```bash
pip install wrds
```

```python
import wrds

db = wrds.Connection(wrds_username="your_username")   # asks for your password the first time

db.list_libraries()                 # what NYCU can see
db.list_tables(library="crsp")

# daily data for Apple (PERMNO 14593), 2017
df = db.raw_sql("""
    select permno, date, prc, ret, vol
    from crsp.dsf
    where permno = 14593
      and date between '2017-01-01' and '2017-12-31'
""", date_cols=["date"])

db.close()
df.head()
```

**Never type your password into a notebook or a script.** Let `wrds.Connection` ask for it.
A password committed to GitHub stays in the history even after you delete the line.

---

## 4. Licence rules: what you may and may not do

WRDS data is licensed to NYCU. You may use it for your coursework and research; you may **not** pass it on.

- Keep downloads in your project's raw-data folder, exactly as downloaded.
- A repository that holds WRDS data must be **private**. Never put WRDS data in a public repository,
  a shared drive or a chat group.
- Commit the **query**: the script, or a note of the four form choices. Anyone with their own WRDS
  account can then rebuild your data. That is what makes a replication reproducible.
- GitHub rejects files over 100 MB. For a large download, commit the query and a small sample.
- In your write-up, state the database, the table, the date range and the date you downloaded.

---

## 5. When the data you need is not on WRDS

Some papers use data that WRDS does not carry. Bloomberg is the common case: futures, FX,
fixed income and many international series are often taken from a Bloomberg Terminal, which is a
separate licence.

What to do, in this order:

1. **Search WRDS first.** Use the search box and the concept groups; a grey database means NYCU
   does not subscribe to it.
2. **Ask whether NYCU has a terminal.** Ask the NYCU Library or the department office; access
   is usually on a specific machine, by appointment.
3. **Use a free substitute and say so.** Example: continuous commodity futures from Yahoo Finance
   (tickers such as `CL=F` for crude oil, `GC=F` for gold) instead of Bloomberg generic contracts.
   Free continuous series roll from one contract to the next in their own way, so returns around
   roll dates can differ from the paper's. State the source, the tickers and how the series is
   rolled, and compare your summary statistics with the paper's table before you model anything.
4. **Ask your supervisor** before you commit to a paper whose data you cannot get.

Taiwan market data is in **TEJ**, which NYCU also subscribes to.
