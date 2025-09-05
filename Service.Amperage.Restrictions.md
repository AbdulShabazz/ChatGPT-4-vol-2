## OpenAI ChatGPT v1.2025.238 

**Q: Is it possible to arbitrarily resize a residential service power/amperage supply/breakers?**

**A:** No. You cannot arbitrarily upsize or downsize a home’s service amperage or main breaker. The rating must satisfy NEC load calculations, minimum service-disconnect requirements, equipment and conductor limits, and your utility’s service rules, with permits and inspections. ([Mike Holt Enterprises][1], [siouxcenter.org][2], [ECM Web][3], [Con Edison][4])

### What governs the service size

* **Minimum allowed:** A one-family dwelling’s service disconnect may not be less than **100 A, 3-wire**. Some jurisdictions amend this to **200 A** for new single-family homes. ([siouxcenter.org][2], [schaumburg.novusagenda.com][5])
* **Calculated load:** The service disconnect must be **≥ the Article 220 calculated demand** and not less than 230.79(A)–(D). Use the dwelling **optional method (NEC 220.82)**: 3 VA/ft² general load, first 10 kVA at 100%, remainder at 40%, then add the larger of heat or A/C. ([Mike Holt Enterprises][6])

### Conductors and equipment must match

* **Service/feeder conductors (dwelling rule):** NEC **310.12** permits sizing at **83% of the service rating** (when no adjustment/correction applies). Typical 120/240 V examples: **200 A → 2/0 Cu or 4/0 Al**. ([Electrical License Renewal][7], [Mike Holt Enterprises][8], [ECM Web][9])
* **Panelboard protection:** The upstream OCPD (main breaker) **cannot exceed the panelboard/bus rating**; using a **smaller** main in a higher-rated panel is permitted. ([ECM Web][3])
* **Branch/feeder OCPD sizing:** For **continuous loads**, the OCPD must be **≥ 125% of the continuous load + 100% of non-continuous** (NEC 210.20(A), 215.3). ([Schneider Electric Product Info][10], [Mike Holt Enterprises][11])

### Utility constraints and workflow

* **You must coordinate with the utility.** Metering type/size and the final live connection are utility-controlled; outdoor metering is often required on 1–3 family homes; standard low-tension metering hardware is specified (e.g., **200 A/400 A cabinets**). ([Con Edison][4])
* **Process:** Licensed electrician + utility work request, utility layout/approval, AHJ permits/inspection, then utility sets the meter and energizes. ([Con Edison][12])

### “Downsizing” or “upsizing” in practice

* **Downsizing** below 100 A is prohibited for a one-family dwelling; downsizing above 100 A is allowed **only** if the **Article 220 demand** supports it and the existing conductors/equipment remain adequately rated. A main breaker swap alone is not lawful if it would **overload** conductors or violate the panel’s listing. ([siouxcenter.org][2], [Mike Holt Enterprises][6], [ECM Web][3])
* **Upsizing** typically requires new service-entrance conductors, meter socket/cabinet per utility spec, grounding/bonding verification, and possibly a new panel. Expect permits, inspections, and a utility outage window. ([Con Edison][4])

### Alternatives to a full service upgrade

* **Energy management for EVs and large loads:** NEC **625.42(A)** allows an **Energy Management System** to **limit EVSE load** so feeders/service can be sized to the EMS setpoint, often avoiding a panel/service upgrade. ([Electrical License Renewal][13], [Mike Holt Enterprises][14])

---

#### Sources (text links)

1. Mike Holt, “Services, based on the 2023 NEC – Part 2” — [https://www.mikeholt.com/newsletters.php?action=display\&letterID=2787](https://www.mikeholt.com/newsletters.php?action=display&letterID=2787) ([Mike Holt Enterprises][1])
2. Weld County (CO) handout citing NEC 230.79(C) — [https://www.weld.gov/files/sharedassets/public/v/5/departments/building/documents/handouts/residential/basic-wiring-requirements.pdf](https://www.weld.gov/files/sharedassets/public/v/5/departments/building/documents/handouts/residential/basic-wiring-requirements.pdf) ([Weld][15])
3. Village of Schaumburg (IL) NEC amendment requiring 200 A for new SFD — [https://schaumburg.novusagenda.com/AgendaPublic/AttachmentViewer.ashx?AttachmentID=11875\&ItemID=6237](https://schaumburg.novusagenda.com/AgendaPublic/AttachmentViewer.ashx?AttachmentID=11875&ItemID=6237) ([schaumburg.novusagenda.com][5])
4. Mike Holt, “Dwelling Load Calculations (NEC 220.82 optional method)” — [https://www.mikeholt.com/newsletters.php?action=display\&letterID=2434](https://www.mikeholt.com/newsletters.php?action=display&letterID=2434) ([Mike Holt Enterprises][6])
5. Electrical License Renewal, “NEC 310.12 Single-Phase Dwelling Services and Feeders (83% rule)” — [https://www.electricallicenserenewal.com/Electrical-Continuing-Education-Courses/NEC-Content.php?sectionID=878](https://www.electricallicenserenewal.com/Electrical-Continuing-Education-Courses/NEC-Content.php?sectionID=878) ([Electrical License Renewal][7])
6. Mike Holt, Table 310.12 excerpt (shows **200 A → 2/0 Cu or 4/0 Al**) — [https://www.mikeholt.com/files/PDF/20EP\_Table\_310.12.pdf](https://www.mikeholt.com/files/PDF/20EP_Table_310.12.pdf) ([Mike Holt Enterprises][8])
7. EC\&M, “NEC Requirements for Switchboards and Panelboards” (NEC 408.36) — [https://www.ecmweb.com/national-electrical-code/code-basics/article/55247439/nec-requirements-for-switchboards-and-panelboards](https://www.ecmweb.com/national-electrical-code/code-basics/article/55247439/nec-requirements-for-switchboards-and-panelboards) ([ECM Web][3])
8. Schneider Electric note on NEC 210.20(A) (125% continuous) — [https://www.productinfo.schneider-electric.com/na-std-ref/5c0fee4b347bdf0001de4d55/%24/NECInformationOfNote-F20BDBCC](https://www.productinfo.schneider-electric.com/na-std-ref/5c0fee4b347bdf0001de4d55/%24/NECInformationOfNote-F20BDBCC) ([Schneider Electric Product Info][10])
9. Con Edison “Electric Blue Book” (service specs, metering, final connection) — [https://www.coned.com/-/media/files/coned/documents/small-medium-large-businesses/electricbluebook.pdf](https://www.coned.com/-/media/files/coned/documents/small-medium-large-businesses/electricbluebook.pdf) ([Con Edison][4])
10. Con Edison Building/Remodeling FAQ (workflow, licensed contractor, approvals) — [https://www.coned.com/en/small-medium-size-businesses/building-project-center/faq](https://www.coned.com/en/small-medium-size-businesses/building-project-center/faq) ([Con Edison][12])
11. Electrical License Renewal, “NEC 625.42 Rating (EV load management)” — [https://www.electricallicenserenewal.com/Electrical-Continuing-Education-Courses/NEC-Content.php?sectionID=1534](https://www.electricallicenserenewal.com/Electrical-Continuing-Education-Courses/NEC-Content.php?sectionID=1534) ([Electrical License Renewal][13])
12. Mike Holt, “Electric Vehicle Power Transfer System” (625.42 EMS example) — [https://www.mikeholt.com/files/PDF/23\_SOLAR\_Article\_625.pdf](https://www.mikeholt.com/files/PDF/23_SOLAR_Article_625.pdf) ([Mike Holt Enterprises][14])

**Bottom line:** You can only “resize” service amperage to the **nearest standard rating that your calculated load, equipment, conductors, and utility rules all support**, and you must document it for permits and the utility. ([Mike Holt Enterprises][1], [Con Edison][4])

[1]: https://www.mikeholt.com/newsletters.php?action=display&letterID=2787&utm_source=chatgpt.com "Services, based on the 2023 NEC - Part 2"
[2]: https://www.siouxcenter.org/DocumentCenter/View/246/GENERAL-ELECTRICAL-REQUIREMENTS-?bidId=&utm_source=chatgpt.com "GENERAL ELECTRICAL REQUIREMENTS FOR SINGLE ..."
[3]: https://www.ecmweb.com/national-electrical-code/code-basics/article/55247439/nec-requirements-for-switchboards-and-panelboards?utm_source=chatgpt.com "NEC Requirements for Switchboards and Panelboards"
[4]: https://www.coned.com/-/media/files/coned/documents/small-medium-large-businesses/electricbluebook.pdf "Specifications for Electric Installations"
[5]: https://schaumburg.novusagenda.com/AgendaPublic/AttachmentViewer.ashx?AttachmentID=11875&ItemID=6237&utm_source=chatgpt.com "CHAPTER 152: - ELECTRICAL CODE Section Footnotes: --- (2)"
[6]: https://www.mikeholt.com/newsletters.php?action=display&letterID=2434 "Dwelling Load Calculations, based on the 2020 NEC"
[7]: https://www.electricallicenserenewal.com/Electrical-Continuing-Education-Courses/NEC-Content.php?sectionID=878 "310.12 Single-Phase Dwelling Services and Feeders."
[8]: https://www.mikeholt.com/files/PDF/20EP_Table_310.12.pdf?utm_source=chatgpt.com "DWELLING UNIT CALCULATIONS"
[9]: https://www.ecmweb.com/national-electrical-code/code-basics/article/55277245/nec-requirements-for-conductors?utm_source=chatgpt.com "NEC Requirements for Conductors"
[10]: https://www.productinfo.schneider-electric.com/na-std-ref/5c0fee4b347bdf0001de4d55/Conductor%20Ampacity%20Tables/English/Data%20Bulletin%20-%20Conductor%20Ampacity%20Tables%20%28bookmap%29_0000264293.ditamap/%24/NECInformationOfNote-F20BDBCC?utm_source=chatgpt.com "NEC Information of Note - North America Standards ..."
[11]: https://www.mikeholt.com/newsletters.php?action=display&letterID=1742&utm_source=chatgpt.com "Article 215 - Feeder Conductors, based on the 2017 NEC"
[12]: https://www.coned.com/en/small-medium-size-businesses/building-project-center/faq?utm_source=chatgpt.com "Building & Remodeling: Frequently Asked Questions | Con Edison"
[13]: https://www.electricallicenserenewal.com/Electrical-Continuing-Education-Courses/NEC-Content.php?sectionID=1534&utm_source=chatgpt.com "625.42 Rating."
[14]: https://www.mikeholt.com/files/PDF/23_SOLAR_Article_625.pdf?utm_source=chatgpt.com "ELECTRIC VEHICLE POWER TRANSFER SYSTEM"
[15]: https://www.weld.gov/files/sharedassets/public/v/5/departments/building/documents/handouts/residential/basic-wiring-requirements.pdf?utm_source=chatgpt.com "Basic Requirements for Wiring a Single Family Dwelling"
