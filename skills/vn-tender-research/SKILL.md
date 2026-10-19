---
name: vn-tender-research
description: Find and summarize Vietnamese public procurement data (procurement plans/KHLCNT, open tender notices, contract award results) from Vietnam's national e-procurement system VNEPS (muasamcong.mpi.gov.vn). Use when the user asks about tenders, bids, procurement plans, contract awards or winning contractors in Vietnam, in English or Vietnamese ("gói thầu", "đấu thầu", "kế hoạch lựa chọn nhà thầu", "kết quả trúng thầu").
---

# Vietnam tender research

This skill uses the Apify Actor `donerightlabs/tender-intelligence` through the plugin's MCP server (`apify-vn-procurement`, hosted by Apify at `mcp.apify.com`). The tool for this Actor has `tender-intelligence` in its name.

## Costs: always confirm before running
The Actor is **pay-per-event**, and the charges go to **the user's own Apify account**. Before every run, show the estimated cost and ask the user to confirm. Never start a paid run without that confirmation. The Actor's Pricing tab is the source of truth; at the time of writing the rates were:
- $0.05 per run (every mode);
- $0.50 per keyword in `notices`, `plans` and `awards` modes (charged even if nothing is found);
- $0.50 once per run in `awards_by_date` mode (no keywords);
- plus $0.05 per package that actually has a published award result, in `awards` and `awards_by_date` modes only (packages with no result yet are not charged this fee).

Lower rates apply on higher Apify plans.

Examples:
- 2 keywords in `notices` mode ≈ $0.05 + 2 × $0.50 = **$1.05**.
- 1 keyword in `awards` mode returning 5 awarded packages ≈ $0.05 + $0.50 + 5 × $0.05 = **$0.80**.
- `awards_by_date` for one province returning 8 awarded packages ≈ $0.05 + $0.50 + 8 × $0.05 = **$0.95**.

Quote the maximum (every requested result awarded) and say it may end up lower.

## Steps
1. **Clarify what the user needs.** Map the request to one `recordType`:
   - `notices`: open invitations to bid (default).
   - `plans`: procurement plans (KHLCNT). These are upcoming purchases, published **before** the tender notice, so they give the earliest signal.
   - `awards`: award results (winning contractors and winning prices) for **closed** packages that match the keywords. Needs keywords. Packages that are closed but have no published result yet come back as a status note, not as an award.
   - `awards_by_date`: award results for one province over a date range, **no keywords needed** (ignore the `keywords` field). This mode **requires** `province`, an exact value from the Actor's list in its input schema (current names after the 2025 merger, for example `Thành phố Hà Nội`, `Tỉnh Gia Lai`). `dateFrom`/`dateTo` (`dd/mm/yyyy`) are optional and default to yesterday → today; set them only when the user wants another range. A busy province can have hundreds of results per day, so keep `maxResultsPerKeyword` modest and offer to narrow the dates.
2. **Write the keywords in Vietnamese.** The portal searches Vietnamese text. Translate English requests into short, common Vietnamese terms, for example "medical equipment" → `thiết bị y tế`, "road repair" → `sửa chữa đường`. Use 1–3 keywords per run, because each keyword adds time (about 1–2 minutes) and cost.
3. **Set limits.** `maxResultsPerKeyword` defaults to 3; raise it only if the user wants more (maximum 50). `onlyOpenTenders` defaults to `true`; set it to `false` for history or trend questions.
4. **Confirm the cost** (see above), then run the Actor.
5. **Read the output.**
   - Records with a `trang_thai` field are status notes, not tenders:
     - `loi_ky_thuat`: a technical error for that keyword. Suggest retrying later.
     - `co_the_con_du_lieu`: the result limit was reached and more data may exist.
     - `chua_co_ket_qua`: the package is closed but no award result is published yet. Say so; do not guess a winner.
     - `khong_co_goi_da_dong`: no closed package matched the keyword, so there is nothing to check for awards.
     - `khong_co_ket_qua_trong_khoang_ngay`: no award results in that province and date range.
     - `khong_doc_duoc_ket_qua`: the result page could not be read. Suggest retrying later.
     - `ghi_chu` explains the situation in Vietnamese. Records of normal results have `trang_thai` empty (`null`).
   - Main fields:
     - `ma_thong_bao`: notice number;
     - `ten_goi_thau`: package name;
     - `ngay_dong_thau`: bid closing time;
     - `dia_diem`: location;
     - `linh_vuc_dau_tu`: field code: `HH` = goods (hàng hoá), `XL` = construction (xây lắp), `TV` = consulting (tư vấn), `PTV` = non-consulting services (phi tư vấn). Show any other code as returned instead of guessing what it means;
     - plans only:
       - `ten_ke_hoach`: plan name;
       - `chu_dau_tu`: investor;
       - `tong_muc_dau_tu_vnd`: total investment in VND;
       - `ngay_phe_duyet`: approval date;
       - `goi_thau_du_kien`: the planned packages, each with name, estimated price, field and location;
     - awards only: `nha_thau_trung_thau`, the winning contractors (name, tax ID `ma_so_thue`, winning price `gia_trung_thau_vnd` in VND, and `thoi_gian_thuc_hien_*` execution times, which can be `null`). A package can have several winners (joint ventures). When a winner looks like an individual rather than an organization, the Actor redacts the name and IDs as `[ĐÃ ẨN...]`: report it as "a winner whose identity is withheld (individual)" and give the price; never try to work out who it is.
     - `awards_by_date` records have `tinh_thanh` (province) instead of `tu_khoa`.
6. **Summarize for the user** in their language: a short table (package, buyer/location, closing date or winner and price) plus 2–3 insights, such as what closes soonest or which contractors win most often. Keep the original Vietnamese names alongside any translation.

## Rules
- Report only what the Actor returned; do not invent tenders, prices or winners. Say so when a keyword returns nothing.
- The data is public procurement information from VNEPS. Present it as information, not as legal or bidding advice, and recommend checking the official notice on muasamcong.mpi.gov.vn before acting.
- Do not add personal contact details (names, phone numbers, emails of individuals) to summaries, even if they appear in free-text fields. The Actor masks phone numbers and emails it detects (`[ĐÃ CHE SĐT]` / `[ĐÃ CHE EMAIL]`), but masking is pattern-based; if you spot anything personal that slipped through, leave it out.
- Do not try to re-identify contractors whose details are redacted (`[ĐÃ ẨN...]`).
- For questions about the Actor itself, point to https://apify.com/donerightlabs/tender-intelligence and its Issues tab.
