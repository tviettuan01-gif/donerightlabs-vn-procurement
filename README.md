# Vietnam Procurement Monitor (donerightlabs)

Search **Vietnam's national e-procurement system** (VNEPS, muasamcong.mof.gov.vn) directly from Claude. Ask in English or Vietnamese, and Claude finds:

- **Procurement plans** (KHLCNT): upcoming purchases, published before the tender notice;
- **Open tender notices**: package name, closing date, location, field;
- **Contract award results**: winning contractor organizations and winning prices in VND.

Claude then summarizes the results for you. The plugin is useful for suppliers, contractors, consultants and market researchers who follow Vietnamese public-sector demand.

## How it works
The plugin adds one skill (`vn-tender-research`) and one remote MCP server:

| Component | What it does |
|---|---|
| `skills/vn-tender-research` | Tells Claude how to choose the right mode, write Vietnamese keywords, confirm costs and read the results |
| MCP server `apify-vn-procurement` → `https://mcp.apify.com?tools=donerightlabs/tender-intelligence` | Apify's hosted MCP server, limited to the Actor [`donerightlabs/tender-intelligence`](https://apify.com/donerightlabs/tender-intelligence). Apify also publishes this server as a connector in the Claude directory |

The plugin itself contains no code, stores no data and sends nothing anywhere except through this MCP server.

## Requirements and costs
- An **Apify account**. You sign in with Apify's own login (OAuth) the first time Claude connects. The plugin never sees your credentials.
- The Actor is **pay-per-event**, and charges go to **your Apify account**. The skill asks Claude to show the estimated cost and get your confirmation before every run. The [Pricing tab](https://apify.com/donerightlabs/tender-intelligence) is the source of truth; at the time of writing the rates were:
  - $0.05 per run;
  - $0.15 per keyword (notices, plans or award results by keyword);
  - $0.15 per run for award results by province and date range (no keyword);
  - $0.03 per package whose award result is read in full.
  - Charges apply only after a search succeeds: a run that ends in a technical error is not charged for the search (Apify's automatic start event, about $0.0002, still applies).

## Example prompts
- "Find open Vietnamese government tenders for medical equipment and list the three closing soonest."
- "Tìm kế hoạch lựa chọn nhà thầu liên quan đến 'camera giám sát' và tóm tắt cho tôi."
- "Who won recent construction tenders in Đà Nẵng last week, and at what prices?"

## Data and privacy
- **What is sent:** your search parameters (keywords, mode, province, dates), from Claude to Apify's MCP server, which runs the Actor in **your** Apify account.
- **What comes back:** public procurement records published on VNEPS. Award results list the winning contractors and their tax IDs. The Actor automatically redacts winners that look like individuals (12-digit personal ID as tax code, or a personal name with no organization marker) and masks phone numbers/emails it finds in free-text fields. This is pattern-based, so it cannot guarantee that no personal detail ever appears; the skill also tells Claude to leave any such detail out of its summaries.
- **Storage:** run inputs and results are stored in your Apify account under Apify's retention settings. donerightlabs does not access your runs unless you share them.
- Privacy policy and terms: https://donerightlabs.tviettuan01.workers.dev/chinh-sach
- Apify's privacy policy: https://docs.apify.com/legal/privacy-policy

## Support
Report issues on the Actor's Issues tab: https://apify.com/donerightlabs/tender-intelligence/issues

## Disclaimer
This is an independent tool by donerightlabs. It is not affiliated with or endorsed by any Vietnamese government agency or by Apify. Results are information, not legal or bidding advice. Check the official notice on muasamcong.mof.gov.vn before acting.

## License
MIT. See [LICENSE](LICENSE).
