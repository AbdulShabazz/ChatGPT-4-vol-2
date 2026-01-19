## OpenAI ChatGPT v1.2026.006 EV.Manufacturer.Charging.Whitelist.Specifications.And.Policy.md

**Q: Where do ev manufacturers specify white listed charging devices and infrastructure?**

**A:** Mostly in the documents that define **compatibility, warranty coverage, and safety requirements**—not in a single “whitelist” page.

Where to look (in order of how often it appears):

1. Owner’s manual / in-vehicle “Charging” manual

* Sections titled “Charging,” “AC charging,” “DC fast charging,” “Using public charging,” “EVSE requirements,” “Adapters.”
* Language is usually **standards-based** (“must be UL-listed,” “J1772,” “CCS,” “NACS,” “IEC 61851”), not brand-name whitelists.
* Often includes explicit exclusions (e.g., “do not use ungrounded outlets,” “no extension cords,” “no modified adapters,” etc.).

2. Warranty booklet and exclusions

* Charging-related damage coverage is typically bounded by:

  * Use of **approved/compatible equipment**
  * **Improper installation** (non-code-compliant circuits, undersized wiring, no GFCI where required, etc.)
  * Use of **non-certified** or **modified** EVSE (Electric Vehicle Supply Equipment)
* This is where manufacturers most clearly draw a line that functions like a whitelist (even if it’s written as “requirements”).

3. Official accessories catalog / parts store

* The only “hard whitelist” many OEMs provide is the set of **OEM-sold EVSEs and adapters** (portable EVSE, wallbox, NACS↔CCS adapters, etc.).
* If it’s sold under the OEM part number, it is implicitly approved.

4. Installation guides and partner wallbox pages

* Many OEMs publish a specific **home charger installation guide** and/or point to a partner (e.g., a co-branded wallbox).
* These pages commonly include explicit statements like “recommended” or “approved for use with [model].”

5. Terms of service for charging networks (brand-operated networks)

* If the automaker operates or bundles network access, “approved use” can be defined there:

  * Plug-and-charge requirements
  * Account linking / authentication conditions
  * Adapter eligibility
* This is more about **authorization** than electrical compatibility.

6. Technical specs pages and regulatory/safety compliance notes

* Sometimes the only “whitelist” is: “Use EVSE certified to **UL 2594 / UL 2231** (North America) or relevant IEC standards (EU), installed to electrical code.”
* OEMs prefer this because it scales across markets.

What “whitelisting” usually means in practice

* Electrical compatibility is governed by **connector + protocol** (J1772 for AC; CCS or NACS for DC in North America) and safety standards.
* Actual brand/model whitelists are uncommon, except for:

  * OEM-branded EVSE and adapters
  * Specific “recommended home charger” partnerships
  * Vehicle-side DC fast charge behavior tied to network features (plug-and-charge), not basic charging.

If you tell me the OEM/model/year and region (e.g., “2025 GM Ultium, U.S.”), I can point to the exact document types/sections where that OEM states its requirements (and whether they use brand-name approvals vs standards language).
