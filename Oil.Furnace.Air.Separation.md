## OpenAI ChatGPT v1.2025.217 

**Q: How does an oil furnace separate the interior heated air from the exhaust during combustion?**

**A:** An oil-fired furnace is engineered so that the interior heated air (the supply air that circulates through your ducts) never physically mixes with the exhaust gases from combustion. This separation is achieved through a **sealed, heat-exchanger-based design**.

---

## 1. **Primary Components Involved**

* **Combustion Chamber**

  * A metal enclosure where fuel oil is atomized, ignited, and burned.
  * Fully isolated from the building’s circulating air.
  * Typically lined with refractory material to withstand \~1,800–2,200 °F combustion temperatures.

* **Heat Exchanger**

  * A welded or cast metal structure forming a barrier between hot combustion gases and the supply air path.
  * The outside surface transfers heat to air, while the inside surface contains exhaust gases.
  * Constructed from steel or stainless steel to resist corrosion from acidic condensates.

* **Air Plenum and Blower**

  * Pulls in cool return air from the building.
  * Forces this air across the *outside* surfaces of the heat exchanger.
  * Heated air is then pushed into the ductwork for distribution.

* **Flue and Draft System**

  * Carries combustion gases from the *inside* of the heat exchanger to the chimney or vent.
  * May include a draft inducer or barometric damper to maintain correct pressure.

---

## 2. **How Separation is Maintained**

* **Physical Barrier**

  * The heat exchanger walls are continuous metal with welded seams or cast sections.
  * No holes or openings connect the air side to the gas side (except in case of damage).

* **Pressure Differential**

  * Combustion side operates at a slightly negative or controlled draft pressure, so if a pinhole forms, exhaust tends to be drawn toward the flue, not into the supply air.
  * Supply-air side is under positive pressure from the blower.

* **Independent Flow Paths**

  * Supply air flows *around* the heat exchanger.
  * Exhaust flows *through* the inside of the exchanger and then out the vent.

---

## 3. **Failure Modes That Risk Mixing**

Even though the system is designed to keep air and exhaust separate, certain faults can cause cross-contamination:

* **Cracked Heat Exchanger** — Corrosion, thermal fatigue, or manufacturing defects can create holes, letting exhaust gases leak into supply air.
* **Improper Draft Control** — Poor venting can reverse pressure differential, pushing exhaust toward the air side.
* **Blocked Flue** — Forces gases to find another path out, potentially through exchanger breaches.

---

## 4. **Safety Considerations**

* **Carbon Monoxide (CO) Detectors** — Required in living spaces with oil or gas combustion appliances.
* **Annual Inspection** — Visual, smoke test, or combustion gas analysis to detect exchanger cracks.
* **Proper Venting** — Ensures combustion gases always exit via designed exhaust paths.

---

If you like, I can diagram the **air vs. exhaust pathways** in a typical oil furnace so you can see exactly where the separation occurs. This would make the design easier to visualize.

**Q: Could stone or rock suffice as a more effective heat exchanger? Could the remaining exhaust chamber be lined with ceramics?**

**A:** Short answer: for the *heat-exchanger itself*, stone/rock is generally a poor choice; for the *combustion/exhaust liner*, ceramics (refractory) are standard practice and often ideal. Details below.

# 1) Stone/rock as a heat-exchanger material — why it underperforms

* **Thermal conductivity (k) — low for rock, high for metals.** Typical granites/basalts: \~2–4 W/m·K; limestone/sandstone can be similar. By contrast: stainless steels \~14–25 W/m·K, mild steel \~50 W/m·K, aluminum \~160 W/m·K, copper \~200 W/m·K. Lower k means thicker walls or greater area is needed to pass the same heat flow, raising size, mass, and cost. ([Wikipedia][1], [Engineering ToolBox][2])
  Textual refs: en.wikipedia.org/wiki/List\_of\_thermal\_conductivities ; engineeringtoolbox.com/thermal-conductivity-metals-d\_858.html

* **Thermal shock & fracture risk — high for most rocks.** Heat exchangers see steep gradients (flame side hot, air side cool) and cycling; many stones/ceramics crack or spall under thermal shock unless specially engineered (microstructure, porosity, fiber reinforcement). ([ScienceDirect][3], [SpringerLink][4])
  Textual refs: sciencedirect.com/topics/materials-science/thermal-shock-resistance ; link.springer.com/chapter/10.1007/978-1-4615-7014-1\_20

* **Manufacturability & geometry — unfavorable.** Effective exchangers use *thin, high-area* walls (tubes, plates, fins). Rock is difficult to form into leak-tight, thin, complex passages, cannot be welded, and is brittle in brazed/bolted assemblies. Metals (stainless, carbon steel, Al, Cu) dominate due to fabricability and strength, despite corrosion trade-offs. ([Fictiv][5], [Nickel Institute][6])
  Textual refs: fictiv.com/articles/heat-exchanger-material-selection ; nickelinstitute.org/media/4658/ni\_aisi\_9005\_heatexchangers.pdf

* **Net effect in an oil furnace.** A stone exchanger would be bulky (low k), slow to warm (high thermal mass), prone to cracking (thermal shock), and hard to seal. Result: *lower steady-state efficiency and poor transients* versus a steel/stainless exchanger. (Data above support each failure mode.) ([Wikipedia][1], [Engineering ToolBox][2], [ScienceDirect][3])
  Textual refs: en.wikipedia.org/wiki/List\_of\_thermal\_conductivities ; engineeringtoolbox.com/thermal-conductivity-metals-d\_858.html ; sciencedirect.com/topics/materials-science/thermal-shock-resistance

> Exception note: advanced ceramics like **aluminum nitride (AlN)** can rival Al in thermal conductivity (≈140–180 W/m·K) while being electrically insulating, but they are expensive and brittle, and are not used as bulk furnace exchangers. ([Thermtest][7])
> Textual ref: thermtest.com/thermal-resources/top-10-resources/top-10-thermally-conductive-materials

# 2) Ceramics to line the *combustion/exhaust* chamber — yes, that’s normal

* **Refractory linings are standard** in burners, boilers, and furnaces: they protect metal shells, shape the flame, cut radiation losses, and withstand 1,000–1,600 °C exposure. Castables, firebrick, and ceramic-fiber blankets are common. ([heatingny.com][8], [thermalprocessing.com][9], [ScienceDirect][10])
  Textual refs: heatingny.com/2021/05/03/the-importance-of-refractory-in-a-boiler/ ; thermalprocessing.com/an-overview-of-refractory-ceramic-fibers/ ; sciencedirect.com/topics/materials-science/refractory-ceramics

* **Why ceramics here work well.** You *want* low thermal conductivity at the flame envelope to keep the hot zone hot and protect steel shells; the actual heat transfer to air occurs *downstream at the metal heat-exchanger surfaces*. Refractory also resists chemical attack from sulfur compounds present in oil-burner flue gas. ([thermalprocessing.com][9])
  Textual ref: thermalprocessing.com/an-overview-of-refractory-ceramic-fibers/

* **Acidic condensate reality in oil appliances.** Oil flue gas can form **sulfuric acid** upon cooling; dew-point can be \~130–150 °C depending on SO₃ and moisture. If metal surfaces fall below this, severe corrosion occurs—one reason true *condensing* oil furnaces are rare and why special alloys/coatings or ceramic linings are used in harsh zones. ([Nippon Steel][11], [ScienceDirect][12], [inspectioneering.com][13])
  Textual refs: nipponsteel.com/product/catalog\_download/pdf/A012en.pdf ; sciencedirect.com/science/article/pii/S2666790820300240 ; inspectioneering.com/journal/2005-05-01/3483/99-diseases-of-pressure-equipm

# 3) Practical material split for an oil furnace

* **Combustion chamber / burner throat:** refractory brick/castable or ceramic-fiber modules to handle flame radiation and damp thermal shock; keep metal shells cool. ([heatingny.com][8], [thermalprocessing.com][9])
  Textual refs: heatingny.com/2021/05/03/the-importance-of-refractory-in-a-boiler/ ; thermalprocessing.com/an-overview-of-refractory-ceramic-fibers/

* **Heat-exchanger (flue-gas side vs. house-air side):** corrosion-resistant metals. Common: 304/316 stainless, duplex grades where needed; specialty “sulfuric-acid dew-point” steels (e.g., ND steel / 09CrCuSb, S-TEN) where condensation risk exists; geometry is thin-wall tubes/plates for high UA. ([Enerquip Thermal Solutions][14], [Nickel Institute][6], [vrcoolertech.com][15])
  Textual refs: enerquip.com/heat-exchanger-material-selection/ ; nickelinstitute.org/media/4658/ni\_aisi\_9005\_heatexchangers.pdf ; vrcoolertech.com/news/how-to-choose-the-material-of-the-heat-recover-77909392.html

* **Why not “all-ceramic” exchangers?** Most ceramics (except niche AlN/SiC) have low k (≈2–30 W/m·K), are difficult to join leak-tight, and carry thermal-shock risk. Metals give superior heat flux for a given wall thickness and are easier to fabricate/repair. ([Wikipedia][1], [Engineering ToolBox][2], [ScienceDirect][3])
  Textual refs: en.wikipedia.org/wiki/List\_of\_thermal\_conductivities ; engineeringtoolbox.com/thermal-conductivity-metals-d\_858.html ; sciencedirect.com/topics/materials-science/thermal-shock-resistance

# 4) If you *wanted* to push ceramics further

* **Hybrid approach (recommended path).** Keep a refractory-lined combustion zone; use *metal* heat-exchange passages immediately downstream. Add **acid-resistant alloys** or coatings only where gas temperatures can cross the sulfuric-acid dew-point. Manage return-air temperatures and flue dilution to *stay above* corrosive dew-points on metal. ([Nippon Steel][11], [inspectioneering.com][13])
  Textual refs: nipponsteel.com/product/catalog\_download/pdf/A012en.pdf ; inspectioneering.com/journal/2005-05-01/3483/99-diseases-of-pressure-equipm

* **Exotic ceramic inserts only where justified.** Consider SiC or AlN tiles *as sacrificial hot-face elements* if you needed extreme radiation tolerance, but still couple to a metallic exchanger for bulk heat recovery. Expect higher cost and careful mounting to accommodate differential expansion. ([Thermtest][7])
  Textual ref: thermtest.com/thermal-resources/top-10-resources/top-10-thermally-conductive-materials

---

## Bottom line

* **Stone/rock**: Not a good heat-exchanger medium in an oil furnace (low k, brittle, hard to seal/fabricate). Use metals. ([Wikipedia][1], [Engineering ToolBox][2])
  Textual refs: en.wikipedia.org/wiki/List\_of\_thermal\_conductivities ; engineeringtoolbox.com/thermal-conductivity-metals-d\_858.html
* **Ceramic lining of the exhaust/combustion chamber**: Yes, that’s the norm (refractory). It shields metal, stabilizes flame, and resists acid attack; the exchanger proper is metallic and placed downstream. ([heatingny.com][8], [thermalprocessing.com][9], [Nickel Institute][6])
  Textual refs: heatingny.com/2021/05/03/the-importance-of-refractory-in-a-boiler/ ; thermalprocessing.com/an-overview-of-refractory-ceramic-fibers/ ; nickelinstitute.org/media/4658/ni\_aisi\_9005\_heatexchangers.pdf

If you want, I can sketch an *optimized section stack* (refractory hot face → protective transition → stainless/duplex exchanger) with target temperatures and materials so you can evaluate a retrofit concept.

[1]: https://en.wikipedia.org/wiki/List_of_thermal_conductivities?utm_source=chatgpt.com "List of thermal conductivities"
[2]: https://www.engineeringtoolbox.com/thermal-conductivity-metals-d_858.html?utm_source=chatgpt.com "Thermal Conductivity of Metals and Alloys: Data Table & ..."
[3]: https://www.sciencedirect.com/topics/materials-science/thermal-shock-resistance?utm_source=chatgpt.com "Thermal Shock Resistance - an overview"
[4]: https://link.springer.com/chapter/10.1007/978-1-4615-7014-1_20?utm_source=chatgpt.com "Thermal Shock Resistance of Ceramic Materials"
[5]: https://www.fictiv.com/articles/heat-exchanger-material-selection?utm_source=chatgpt.com "Heat Exchanger Material Selection"
[6]: https://nickelinstitute.org/media/4658/ni_aisi_9005_heatexchangers.pdf?utm_source=chatgpt.com "ROLE OF STAINLESS STEELS IN INDUSTRIAL HEAT ..."
[7]: https://thermtest.com/thermal-resources/top-10-resources/top-10-thermally-conductive-materials?utm_source=chatgpt.com "Top 10 Thermally Conductive Materials"
[8]: https://heatingny.com/2021/05/03/the-importance-of-refractory-in-a-boiler/?utm_source=chatgpt.com "The Importance of Refractory in a Boiler"
[9]: https://thermalprocessing.com/an-overview-of-refractory-ceramic-fibers/?utm_source=chatgpt.com "An overview of refractory ceramic fibers"
[10]: https://www.sciencedirect.com/topics/materials-science/refractory-ceramics?utm_source=chatgpt.com "Refractory Ceramics - an overview"
[11]: https://www.nipponsteel.com/product/catalog_download/pdf/A012en.pdf?utm_source=chatgpt.com "Sulfuric Acid and Hydrochloric Acid Dew-point Corrosion- ..."
[12]: https://www.sciencedirect.com/science/article/pii/S2666790820300240?utm_source=chatgpt.com "Role of condensing heat exchanger material"
[13]: https://inspectioneering.com/journal/2005-05-01/3483/99-diseases-of-pressure-equipm?utm_source=chatgpt.com "99 Diseases of Pressure Equipment: Flue Gas Dew Point ..."
[14]: https://www.enerquip.com/heat-exchanger-material-selection/?utm_source=chatgpt.com "Heat Exchanger Material Selection Based on Common ..."
[15]: https://www.vrcoolertech.com/news/how-to-choose-the-material-of-the-heat-recover-77909392.html?utm_source=chatgpt.com "How To Choose The Material Of The Heat Recovery Based ..."

**Q: Could a spiral-shaped exhaust chamber more effectively transfer thermal energy to the copper heat exchanger, compared to using am open cavity?**

**A:** Short answer: yes—routing flue gas through a spiral (curved/helical) passage will generally transfer more heat to a downstream exchanger than an open cavity, because it (i) increases surface area, (ii) raises gas-side turbulence via secondary (Dean) vortices, and (iii) lengthens residence time. The gains come with costs: higher pressure drop, potentially greater ash/soot impingement, and stricter materials/condensation management—especially critical for oil flue gas.

### Mechanism & expected gains (spiral/helical vs. open cavity)

* **Heat-transfer coefficient (gas side)**

  * Curved ducts generate *Dean vortices* that disrupt the boundary layer and raise the Nusselt number (Nu). Reviews and experiments on curved/helical tubes consistently show higher Nu than straight channels at similar Reynolds numbers, with enhancement rising with Dean number $De=Re\sqrt{D/R}$. Typical boosts range from \~1.2× to \~2× depending on geometry and flow regime. ([Taylor & Francis Online][1], [ResearchGate][2], [ASME Digital Collection][3])
* **Residence time & surface area**

  * A spiral path increases effective length and packable area vs. a single open plenum, improving the overall $UA$ (area A × overall coefficient U). Commercial spiral HX literature emphasizes compactness and single-channel flow that maintains velocity and shear, mitigating dead zones common to open cavities. ([alfalaval.com.br][4], [alfalaval.com][5])
* **Fouling behavior**

  * Oil combustion can shed soot/ash. Single-channel spiral paths tend to be more *self-scouring* (high shear across the whole section) compared with many-tube or open spaces where flow can bypass fouled regions; several case studies report lower plugging rates in spiral HX duty. ([alfalaval.com][6])

### Trade-offs you must budget for

* **Pressure drop (Δp)**

  * The same curvature that enhances Nu also increases friction factor; helical/spiral passages commonly show Δp multipliers of \~1.5–3× vs. straight paths at comparable mass flow. Your burner fan/draft system must have margin (or you add an inducer). ([ScienceDirect][7], [Taylor & Francis Online][1])
* **Soot/ash deposition**

  * Secondary flows help, but elbows/turns can also become deposit initiation points if velocities or wall temperatures are low. Design for mechanical/steam access cleaning and maintain gas velocities high enough to keep surfaces swept. ([PMC][8])
* **Condensation/corrosion window**

  * Oil flue gas contains SO₂/SO₃; when surfaces fall below the **sulfuric-acid dew point** (\~110–150 °C depending on SO₃/H₂O), H₂SO₄ forms and aggressively attacks common alloys. Spiral sections that cool “too well” at the cold end risk low-temperature (dew-point) corrosion unless materials and temperatures are managed. ([PMC][9], [Water Technologies][10], [ECOONEX][11])

### About using **copper** on the flue-gas side

* **Not recommended** in oil-fired flue gas streams: copper and common brasses have poor resistance to sulfuric environments; industry practice prefers stainless steels or nickel-containing alloys for surfaces that can see acid condensation. Use copper on the *air side* if desired, but keep the flue-gas side stainless/advanced alloys or coated steels. ([plymouth.com][12], [Enerquip Thermal Solutions][13], [Nickel Institute][14])

### Practical spiral design notes (gas side)

* **Target flow/turbulence**

  * Choose curvature radius $R$ and hydraulic diameter $D$ to hit **Dean numbers** $De \sim 100–400$ at operating Reynolds numbers—strong enough to induce stable secondary vortices without untenable Δp. Start from correlations in curved-tube reviews and verify with a 1D network model. ([Taylor & Francis Online][1], [ResearchGate][2])
* **Thermal duty & wall temperatures**

  * Keep the *coldest* flue-gas-side metal wall above the site-specific H₂SO₄ dew point (use an acid dew-point estimator and your fuel’s sulfur/SO₃); if you must cross it, confine that region to **acid-resistant materials** and design for drainage/washdown. ([Perma Pure][15], [Esteem Projects Pvt. Ltd.][16])
* **Materials**

  * **316/317 stainless** or **Ni-bearing alloys** for cold-end spiral passages; upgrade only where analysis shows risk. Documented corrosion behavior in SOₓ-bearing flue gas favors these over copper alloys. ([Nickel Institute][14])
* **Maintenance**

  * Provide full-diameter cleanouts at spiral ends; avoid sharp mitered turns; maintain minimum gas velocities to limit deposition. Case reports for refinery services show spiral units ran plug-free with improved reliability when sized for shear. ([alfalaval.com][6])

### Quantitative back-of-envelope (for feasibility screening)

* **Heat-transfer gain**: If your current “open cavity” behaves like a low-velocity plenum with effective gas-side $h_g \approx 10–20$ W·m⁻²·K⁻¹, a properly sized spiral duct can plausibly reach $h_g \approx 20–50$ W·m⁻²·K⁻¹ (≈1.5–3×), *plus* a large increase in effective area A—often the dominant factor. Literature on helical/curved channels reports Nu and hence $h$ increases in this range at moderate De. ([Taylor & Francis Online][1], [ResearchGate][2])
* **Pressure-drop penalty**: Expect Δp to rise roughly proportionally with added length and curvature factor; a 1.5–3× Δp increase vs. a straight/high-plenum path is a common planning figure until you run detailed calcs. ([ScienceDirect][7])

### Recommendation (oil furnace context)

1. **Yes, prefer a spiral (or helical) flue-gas path** over an open cavity to boost $UA$, provided the burner/inducer can accommodate the added Δp. Prioritize single-channel spiral geometries that keep velocity uniform and minimize bypass. ([alfalaval.com.br][4])
2. **Do not put copper in contact with flue gas**. Use stainless/Ni alloys for the spiral and keep copper—if used at all—strictly on the clean, *air* side. ([Nickel Institute][14], [Enerquip Thermal Solutions][13])
3. **Control dew-point corrosion** by keeping metal-skin temperatures above the sulfuric-acid dew point, or by limiting cold-end length to acid-resistant materials—with access for inspection/cleaning. ([PMC][9], [Water Technologies][10])

If you want, I can sketch a quick sizing workflow (fuel sulfur → estimated SO₃ → acid dew point → target wall-temperature margin → pick D, R, L for a target Dean number and Δp) with example numbers you can tune to your burner’s draft curve.

[1]: https://www.tandfonline.com/doi/abs/10.1080/01457632.2015.1089735?utm_source=chatgpt.com "A Review of Heat Transfer and Pressure Drop Correlations ..."
[2]: https://www.researchgate.net/publication/281791171_A_Review_of_Heat_Transfer_and_Pressure_Drop_Correlations_for_Laminar_Flow_in_Curved_Circular_Ducts?utm_source=chatgpt.com "(PDF) A Review of Heat Transfer and Pressure Drop ..."
[3]: https://asmedigitalcollection.asme.org/fluidsengineering/article/128/5/1013/477831/Effects-of-Curvature-and-Convective-Heat-Transfer?utm_source=chatgpt.com "Effects of Curvature and Convective Heat Transfer in ..."
[4]: https://www.alfalaval.com.br/globalassets/documents/industries/chemicals/petrochemicals/editorial_consider_spiral_heat_exchangers_ppi00090en.pdf?utm_source=chatgpt.com "Consider spiral heat exchangers for fouling applications"
[5]: https://www.alfalaval.com/products/heat-transfer/plate-heat-exchangers/welded-spiral-heat-exchangers/technical-information/?utm_source=chatgpt.com "Technical information about Welded spiral heat exchangers"
[6]: https://www.alfalaval.com/globalassets/documents/products/heat-transfer/plate-heat-exchangers/welded-spiral-heat-exchangers/spiral-heat-exchanger-solves-costly-fouling-problems_case-story_en.pdf?utm_source=chatgpt.com "Spiral Heat Exchanger solves costly fouling problems"
[7]: https://www.sciencedirect.com/science/article/abs/pii/S0140700718304912?utm_source=chatgpt.com "Experimental and numerical investigations on the heat ..."
[8]: https://pmc.ncbi.nlm.nih.gov/articles/PMC8307845/?utm_source=chatgpt.com "Effect of Flue Gases' Corrosive Components on the ..."
[9]: https://pmc.ncbi.nlm.nih.gov/articles/PMC7758941/?utm_source=chatgpt.com "Investigation on the Corrosion of the Elbows in the Flue ..."
[10]: https://www.watertechnologies.com/handbook/chapter-22-cold-end-deposition-and-corrosion-control?utm_source=chatgpt.com "Chapter 22 - Cold-end Deposition and Corrosion Control"
[11]: https://ecoonex.com/acid-dew-point/?utm_source=chatgpt.com "Sulphuric Acid Dew Point"
[12]: https://www.plymouth.com/wp-content/uploads/2019/08/REVISED-Reliable-Exchanger-Tubing-Janikowski.pdf?utm_source=chatgpt.com "Factors for Selecting Reliable Heat Exchanger ..."
[13]: https://www.enerquip.com/heat-exchanger-material-selection/?utm_source=chatgpt.com "Heat Exchanger Material Selection Based on Common ..."
[14]: https://nickelinstitute.org/media/1829/thecorrosionresistanceofnickel_containingalloysinsulphuricacidandrelatedcompounds_1318_.pdf?utm_source=chatgpt.com "The Corrosion Resistance of Nickel-Containing Alloys in ..."
[15]: https://www.permapure.com/2021/04/09/sulfuric-acid-dew-point-calculator/?utm_source=chatgpt.com "Sulfuric Acid Dew Point Calculator"
[16]: https://www.esteemprojects.com/library/Fired-Heater-Design/page5_6.htm?utm_source=chatgpt.com "Acid Dew Point Of Flue Gas"

**Q: So a spiral chamber or tube, rather than a (narrower) spiral channel.**

**A:** Got it — you’re asking about a **spiral-shaped chamber or tube** (think of a coiled flue passage) as opposed to a *narrow cross-section spiral channel* (like a small passage in a plate heat exchanger).

Functionally, a **spiral tube** behaves more like a *curved duct* in heat transfer textbooks, whereas a *spiral channel* behaves like a shallow groove between plates. The physics are related but not identical.

---

## 1. Spiral Tube (or Chamber) vs. Narrow Spiral Channel

**Spiral Tube or Chamber**

* **Geometry:** A single, continuous, enclosed tube or passage coiled into a spiral shape; diameter is large enough that flue gas flow resembles duct flow, not capillary or microchannel flow.
* **Flow pattern:**

  * If the diameter is fairly large compared to curvature radius, you still get **Dean vortices** but also a more fully-developed internal swirl.
  * This creates *bulk mixing* across the cross-section, which helps even out wall temperature and improves uniform heat pickup by the surrounding heat exchanger.
* **Thermal effect:**

  * Increased **surface area** (longer path length in the same footprint).
  * Increased **turbulence** from curvature-induced vortices, improving gas-side heat transfer coefficient $h_g$.
  * **Residence time** is longer compared to an open plenum, so more total energy is transferred before gases reach the flue outlet.
* **Drawbacks:**

  * Higher **pressure drop** than a straight tube of equal length, but less severe than a very narrow channel.
  * More challenging to mechanically clean if soot builds up, unless cleanout ports are integrated at turns.

**Narrow Spiral Channel**

* **Geometry:** Shallow depth, high width-to-height ratio, usually part of a plate or cast body.
* **Flow pattern:**

  * Higher surface-to-volume ratio per unit length.
  * Stronger shear at walls, but much greater risk of fouling from oil-burner soot due to restricted cross-section.
* **Thermal effect:**

  * Potentially very high $h_g$ per unit area, but absolute gas throughput can be limited by the narrow gap.
* **Drawbacks:**

  * Much higher **pressure drop** for the same volumetric flow.
  * Cleaning is more difficult than in a large tube.

---

## 2. Why a Spiral Tube or Chamber Could Outperform an Open Cavity

An **open cavity** (like a large plenum or box-shaped chamber) is very poor at gas-side heat transfer because:

* Flow tends to **slow down and stratify** — hot gases rise, cooler gases sink, leaving low-velocity zones where heat transfer is weak.
* The effective **heat transfer coefficient $h_g$** can be under 15–20 W/m²·K for oil flue gas in open plenums.
* You get “short-circuit” flow: some gas flows directly to the outlet with minimal contact time.

A **spiral tube or chamber** improves this by:

1. **Maintaining velocity** — consistent duct cross-section means no stagnant zones.
2. **Lengthening path** — more surface area and residence time for a given footprint.
3. **Forcing mixing** — curvature induces cross-sectional swirl, breaking thermal boundary layers.
4. **Targeted heat capture** — allows the metal wall to “see” more consistent hot gas temperature along its length, so heat pickup is more uniform.

---

## 3. Design Considerations for Oil Furnace Use

* **Diameter selection:**
  Large enough to avoid excessive Δp and clogging, but small enough to keep velocity up (typical: 10–20 m/s gas velocity for oil-fired heat recovery without condensing).
* **Material choice:**
  Stainless steel or high-alloy steel on flue-gas side; avoid copper due to sulfuric acid attack risk.
* **Dew point management:**
  Keep cold-end wall temperature above the acid dew point (\~120–150 °C for oil with sulfur) to prevent corrosion.
* **Cleaning access:**
  Design cleanout ports at both ends and possibly midpoints to mechanically sweep soot from coil walls.
* **Spiral pitch & curvature:**
  Tighter coils (smaller radius) increase Dean number and turbulence but also Δp. In practice, a curvature ratio $D/R$ of \~0.05–0.2 gives good enhancement without extreme pressure penalty.

---

## 4. When a Spiral Tube is the Better Option

A **spiral tube** is generally the “sweet spot” for retrofitting an oil furnace to improve heat recovery:

* It **retains cleanability** (rodding/brush access) unlike very narrow channels.
* It **fits in a compact footprint** while adding length and surface area.
* It **boosts heat transfer** over an open plenum by a factor of \~1.5–3× for the same temperature drop, depending on velocity and curvature.

---

If you’d like, I can run a **back-of-envelope thermal gain vs. pressure drop calculation** for a spiral tube coil sized for a typical 100,000 BTU/h oil furnace flue, so we can see whether the trade-offs are worth it in your case. That would give you a concrete efficiency estimate before fabrication.

**Q: What if a microscreen replaces the contiguous walls so that air is Encouraged to flow around the spiral perimeter however gas may flow directly upwards through the microscreens provided they are not blocked by soot. The microscreen is intended to behave as a flow-based overflow valve.**

**A:** Short answer: a **microscreen “wall” is unsafe and code-noncompliant** if it separates the exhaust stream from the building’s circulating air. Heat-exchanger boundaries in oil furnaces must be **gas-tight**; a porous or screen boundary would allow CO/NOx/SO₂-bearing flue gas to leak into the supply air, violating UL/NFPA requirements for oil-fired central furnaces. If you like the “overflow valve” idea, it can only be used **within the flue-gas circuit itself** (i.e., gas-to-gas), not between exhaust and room air. See details and safer variants below.

---

### 1) Code & safety constraints (non-negotiable)

* **Sealed, gastight exchanger boundary:** Oil-fired central furnaces are certified to **UL 727**; the model codes (e.g., IRC M1402.1) require oil furnaces to conform to UL 727. A porous boundary between combustion gas and supply air would not comply. ([ICC Digital Codes][1], [UL Standards Shop][2])
  Source: “IRC 2018, M1402.1: Oil-fired central furnaces shall conform to ANSI/UL 727.” — codes.iccsafe.org/content/IRC2018/chapter-14-heating-and-cooling-equipment-and-appliances ; “UL 727—Oil-Fired Central Furnaces.” — shopulstandards.com/ProductDetail.aspx?UniqueKey=33922
* **Installation standard:** **NFPA 31** governs installation and safety of liquid-fuel appliances; designs are predicated on *no mixing* of combustion products with supply air. A microscreen boundary would undermine that premise. ([NFPA][3])
  Source: “NFPA 31—Standard for the Installation of Oil-Burning Equipment.” — nfpa.org/codes-and-standards/nfpa-31-standard-development/31 ; nfpa.org/product/nfpa-31-standard/p0031code

**Conclusion:** A microscreen replacing the contiguous metal wall **cannot** be used to separate exhaust from interior air.

---

### 2) Engineering reality of microscreens in hot flue gas

* **Pressure drop & fouling:** Flow through woven screens carries a significant Δp that rises with Reynolds number and decreases with porosity; classic and modern correlations quantify this (e.g., Armour & Cannon; Azizi et al.). Fine pores foul quickly with oil-soot/ash, turning your “overflow” into a **choke**. ([AIChE Journal][4], [ScienceDirect][5])
  Source: “Fluid flow through woven screens.” — aiche.onlinelibrary.wiley.com/doi/abs/10.1002/aic.690140315 ; “On the pressure drop of fluids through woven screen meshes.” — sciencedirect.com/science/article/abs/pii/S0009250919305494
* **Unsteady wake formation:** Behind screens, vortices and recirculation zones appear at modest screen-Re, affecting both heat transfer and soot deposition (DLR study). ([DLR eLib][6])
  Source: elib.dlr.de/87260/1/p308.pdf
* **Porous-media combustion ≠ porous boundary to room air:** Porous burners/screens are powerful **within** the combustion/recirculation zone (enhanced mixing, radiant heat release), but they are applied where **both sides are flue gas**—not as leak-paths to clean air. ([ScienceDirect][7], [ResearchGate][8])
  Sources: sciencedirect.com/article/abs/pii/S0306261909000294 ; sciencedirect.com/article/abs/pii/S0016236121012904 ; researchgate.net/publication/372495372\_REVIEW\_OF\_RECENT\_TECHNIQUES\_IN\_POROUS\_MEDIA\_COMBUSTION

---

### 3) Thermo-chemistry constraint you can’t dodge (oil = sulfur)

* **Acid dew point:** Oil flue gas forms **H₂SO₄** on cooling; the sulfuric-acid dew point is typically **\~120–150 °C** depending on SO₃/H₂O. Any “too-effective” local cooling at a screen or cold wall invites **dew-point corrosion**. Materials must be chosen (and temperatures controlled) accordingly. ([SpringerLink][9], [Citizendium][10])
  Sources: Aydemir (2020) notes 135 °C mid-range reference; “Acid dew point” explainer. — link.springer.com/article/10.1007/s40974-020-00171-5 ; en.citizendium.org/wiki/Acid\_dew\_point
* **Mitigation materials:** Where condensation is possible, stainless or specialty low-temperature sulfuric-acid-resistant steels (e.g., S-TEN, 09CrCuSb) are used. ([Nippon Steel][11])
  Source: nipponsteel.com/product/catalog\_download/pdf/A012en.pdf

---

### 4) What *is* feasible: keep the sealed metal wall; add controlled **gas-to-gas** bypass or mixing devices

If your goal is “overflow” behavior to cap Δp and add controllable mixing, do it **inside the flue-gas side only**, while preserving a **continuous, gastight** metal boundary to the supply air:

* **Option A — Spiral tube with *internal* screens as turbulators (gas-side only):**
  Use **coarse** (high-porosity) wire screens or perforated strips mounted inside the spiral tube to raise gas-side $h$ and even out temperature, but keep a **solid stainless shell** between flue gas and air. Size screens to achieve target shear with acceptable Δp (use Armour/Cannon-type correlations; start with porosity > 0.7, wire dia 0.3–0.6 mm; verify Δp against your draft curve). Provide brush/rod access. ([AIChE Journal][4])
  Source: aiche.onlinelibrary.wiley.com/doi/abs/10.1002/aic.690140315

* **Option B — Dual-path *flue-gas* “overflow” (bypass) with a pressure-balanced flap:**
  Build a **sealed** spiral primary path plus a **sealed** straight bypass that re-merges upstream of the flue outlet. A spring-or counterweight-loaded flap opens the bypass when Δp exceeds a setpoint, acting as an *overflow valve* **without** any leakage to the supply air. (This is analogous to draft-control concepts, but kept internal to the HX module.) Conform overall appliance operation to UL 727 test conditions (steady combustion, specified flue stability). ([GovInfo][12])
  Source: 10 CFR § 431.76 references UL 727 test conditions — govinfo.gov/content/pkg/CFR-2013-title10-vol3/pdf/CFR-2013-title10-vol3-sec431-76.pdf

* **Option C — Porous-media *burner* zone upstream, then metal HX:**
  If you want the benefits of porous elements, place them **inside the combustion zone** to stabilize flames and enhance radiant exchange (all on the gas side), then pass products into a conventional **stainless** spiral exchanger. Do **not** use porous elements as the wall to the air stream. ([ScienceDirect][7])
  Source: sciencedirect.com/article/abs/pii/S0306261909000294

---

### 5) Quantitative screening (order-of-magnitude, to guide prototyping)

* **Screen-induced Δp (single layer, air/flue gas at furnace conditions):**
  Empirical forms take $\Delta p \approx \tfrac12\,\rho\,V^2\,K$, with $K = a/Re + b$ depending on mesh geometry/porosity; at furnace-like $Re\sim10^3–10^4$, the inertial term dominates (use literature coefficients; expect **tens to a few hundred Pa** per screen at 10–20 m/s if porosity ≈ 0.7–0.8). Stack-ups multiply quickly. Validate against **Armour & Cannon** curves and modern datasets. ([AIChE Journal][4], [ScienceDirect][5])
  Sources: aiche.onlinelibrary.wiley.com/doi/abs/10.1002/aic.690140315 ; sciencedirect.com/article/abs/pii/S0009250919305494
* **Thermal benefit vs. corrosion risk:**
  Any device that lifts $h_g$ and extends residence time also cools the cold-end metal. Keep the **coldest wall** ≥ **(acid dew point + 10–20 K margin)** or confine sub-dew regions to acid-resistant alloys/coatings. For No. 2 oil, plan around **120–150 °C** dew-point unless you have sulfur/SO₃ analyses. ([SpringerLink][9], [Citizendium][10])
  Sources: link.springer.com/article/10.1007/s40974-020-00171-5 ; en.citizendium.org/wiki/Acid\_dew\_point

---

### 6) Bottom line

* **Do not** use a microscreen as the **wall** between exhaust and supply air. It fails the sealed-exchanger requirement embedded in **UL 727 / IRC / NFPA 31** and presents CO exposure risk. ([ICC Digital Codes][1], [UL Standards Shop][2], [NFPA][3])
  Sources: codes.iccsafe.org/.../IRC2018/chapter-14-heating-and-cooling-equipment-and-appliances ; shopulstandards.com/ProductDetail.aspx?UniqueKey=33922 ; nfpa.org/codes-and-standards/nfpa-31-standard-development/31
* If you want an “overflow-valve” behavior, implement it **inside the flue-gas network** with **sealed** dual paths or **internal** coarse screens/turbulators—**never** as a porous boundary to the air stream—and manage **Δp, fouling, and acid-dew-point** temperatures with the materials noted above. ([GovInfo][12], [AIChE Journal][4], [Nippon Steel][11])
  Sources: govinfo.gov/content/pkg/CFR-2013-title10-vol3/pdf/CFR-2013-title10-vol3-sec431-76.pdf ; aiche.onlinelibrary.wiley.com/doi/abs/10.1002/aic.690140315 ; nipponsteel.com/product/catalog\_download/pdf/A012en.pdf

If you want, I can draft a **sealed spiral HX** bill-of-materials with: (a) stainless grades by temperature zone, (b) target gas velocity & allowable Δp, (c) optional internal screen geometry (porosity/mesh) with cleaning ports, and (d) dew-point control setpoints—so you can sanity-check a prototype against UL 727 operating conditions.

[1]: https://codes.iccsafe.org/content/IRC2018/chapter-14-heating-and-cooling-equipment-and-appliances?utm_source=chatgpt.com "CHAPTER 14 HEATING AND COOLING EQUIPMENT AND ..."
[2]: https://www.shopulstandards.com/ProductDetail.aspx?UniqueKey=33922&utm_source=chatgpt.com "UL 727 - Oil-Fired Central Furnaces - UL Standards"
[3]: https://www.nfpa.org/codes-and-standards/nfpa-31-standard-development/31?utm_source=chatgpt.com "NFPA 31 Standard Development"
[4]: https://aiche.onlinelibrary.wiley.com/doi/abs/10.1002/aic.690140315?utm_source=chatgpt.com "Fluid flow through woven screens - 1968 - AIChE Journal"
[5]: https://www.sciencedirect.com/science/article/abs/pii/S0009250919305494?utm_source=chatgpt.com "On the pressure drop of fluids through woven screen meshes"
[6]: https://elib.dlr.de/87260/1/p308.pdf?utm_source=chatgpt.com "Numerical Investigations on Fluid Flow through Metal ..."
[7]: https://www.sciencedirect.com/science/article/abs/pii/S0306261909000294?utm_source=chatgpt.com "Applications of porous media combustion technology"
[8]: https://www.researchgate.net/publication/372495372_REVIEW_OF_RECENT_TECHNIQUES_IN_POROUS_MEDIA_COMBUSTION?utm_source=chatgpt.com "review of recent techniques in porous media combustion"
[9]: https://link.springer.com/article/10.1007/s40974-020-00171-5?utm_source=chatgpt.com "Estimating excess heat from exhaust gases: why corrosion ..."
[10]: https://en.citizendium.org/wiki/Acid_dew_point?utm_source=chatgpt.com "Acid dew point"
[11]: https://www.nipponsteel.com/product/catalog_download/pdf/A012en.pdf?utm_source=chatgpt.com "Sulfuric Acid and Hydrochloric Acid Dew-point Corrosion- ..."
[12]: https://www.govinfo.gov/content/pkg/CFR-2013-title10-vol3/pdf/CFR-2013-title10-vol3-sec431-76.pdf?utm_source=chatgpt.com "Department of Energy § 431.76 - (2) UL 727 ..."

**Q: Clarification: the microscreen is in the spiral exhaust-chamber only to allow the air to flow directly out of the chamber rather than trace along its spiral during undesirable pressure conditions.**

**A:** Thanks—that helps. Interpreting your idea as: **a porous “microscreen” panel located entirely on the *flue-gas* side** of a spiral exhaust passage, acting as a *pressure-relief overflow* so hot gas can exit more directly when ∆p gets too high—**without** opening any path to the building’s supply air. In that **gas-to-gas** context the concept is feasible, but it’s usually better executed with **coarse, cleanable perforated plates or wire screens** sized from pressure-loss correlations, or with a **sealed bypass flap**, rather than with very fine “micro” meshes that soot will blind rapidly.

Below is a practical, numbers-first way to evaluate and size it.

---

## Feasibility summary (gas-to-gas overflow inside the flue path)

* **Thermal/flow benefit.** The spiral maintains velocity and Dean-vortex mixing; a relief path keeps draft stable by capping ∆p during transient fouling or cold starts. Curved/helical passages consistently show **Nu↑ with curvature ratio and Dean number**, but also **resistance↑**—your relief mitigates that when needed. *Evidence:* curved/spiral literature reports Nu increases and resistance coefficient increases as curvature tightens. See reviews and recent data (e.g., Nu and resistance ↑ with curvature ratio). Source: “Dean number…not sufficient; curvature ratio increases both resistance and Nusselt,” *Applied Thermal Engineering* (2024). URL: sciencedirect.com/article/pii/S2214157X23011607. ([ScienceDirect][1])
* **Fouling risk.** No. 2 oil flue gas carries soot/ash; **fine meshes** load quickly. Spiral HX vendors combat fouling with **single-channel, high-shear geometries** and easy cleaning access; they explicitly market “self-cleaning” via velocity redistribution—useful as a design cue. Source: Alfa Laval welded spiral HX pages. URLs: alfalaval.us/…/welded-spiral-heat-exchangers/ ; alfalaval.com/…/technical-information/ ; alfalaval.us/…/unique-features/ . ([alfalaval.us][2], [alfalaval.com][3])
* **Cold-end corrosion guardrail.** Any enhancement that cools the cold end risks **sulfuric-acid dew-point** condensation; typical **H₂SO₄ dew-points \~120–150 °C** (depend on SO₃ and moisture). Maintain cold-wall ≥ dew-point + margin, or use acid-resistant alloys locally. Sources: Veolia Water Tech “Cold-end deposition,” URL: watertechnologies.com/handbook/chapter-22-cold-end-deposition-and-corrosion-control ; PermaPure calculator, URL: permapure.com/2021/04/09/sulfuric-acid-dew-point-calculator/ ; Nippon Steel S-TEN notes, URL: nipponsteel.com/product/catalog\_download/pdf/A012en.pdf. ([Water Technologies][4], [Perma Pure][5])

> **Safety/code reminder (unchanged):** The **wall between flue gas and house air must remain gastight** per **UL 727** and installation per **NFPA 31**. Your overflow must vent **from gas → gas only**, never to supply air. Sources: UL 727 scope page (UL Standards shop), URL: shopulstandards.com/ProductDetail.aspx?UniqueKey=33922 ; federal IBR to UL 727 in 10 CFR 431.76, URL: law\.resource.org/pub/us/cfr/ibr/006/ul.727.1994.pdf ; NFPA 31 overview, URL: nfpa.org/codes-and-standards/nfpa-31-standard-development/31 . ([UL Standards Shop][6], [Law Resource][7], [NFPA][8])

---

## How to size the **overflow** (two robust options)

### Option A — **Perforated-plate or coarse wire-screen panel** (gas-to-gas bleed into the main flue)

Use standard loss-coefficient correlations so the panel **begins to pass significant flow only when ∆p exceeds a setpoint**. Treat it like a fixed-K check on ∆p.

* **Pressure-loss model (perforated plate):**
  $\Delta p \approx \tfrac{1}{2}\,\rho\,V^2\,K_\mathrm{plate}$, with $K_\mathrm{plate}$ from **Idelchik** (function of **β = open-area ratio**, hole diameter $d$, plate thickness $t$, and layout). Practical tools derive $K$ from Idelchik’s charts/tables. Source: AFT note citing Idelchik (1994), URL: aft.com/support/product-tips/perforated-plates-pressure-drop ; recent model papers: S. Li et al. 2024 (Int. J. Heat Mass Transf.), URL: sciencedirect.com/article/pii/S0017931024003211 ; preprint 2023, URL: arxiv.org/pdf/2304.11730. ([AFT][9], [ScienceDirect][10], [arXiv][11])
  **Starting targets:** β≈0.25–0.40, $t/d \le 1$, round holes on 60° stagger. That typically yields **K ≈ 2–8** (order-of-magnitude), so at flue-gas ρ≈0.5–0.8 kg·m⁻³ and V≈10–20 m·s⁻¹, you can set bleed thresholds in the **tens to a few hundred Pa** range—tune with β and total panel area.

* **Pressure-loss model (coarse woven screen):**
  Classic **Armour & Cannon** correlation (and modern updates) gives $f$ (or $K$) vs. **Re** and **porosity ε**. Use **ε ≥ 0.7** and **wire dia \~0.3–0.8 mm** to avoid rapid blinding; verify ∆p at your gas Re (10³–10⁴ typical). Sources: Armour & Cannon (AIChE J., 1968), URL: aiche.onlinelibrary.wiley.com/doi/abs/10.1002/aic.690140315 ; Wu et al. (2005), URL: sciencedirect.com/article/abs/pii/S0017931005001638 ; Wang et al. (2021), URL: sciencedirect.com/article/abs/pii/S0009250920305698 . ([AIChE Journal][12], [ScienceDirect][13])

* **Fouling control:** Prefer **perforated plate** (millimeter-scale holes) over “micro” meshes. Provide a **vertical orientation** and **brush/rod access**. Expect deposit initiation at leading edges; keep local wall ≥ acid-dew-point (above) to avoid sticky acid films. General fouling references: industrial flue-gas HX fouling design studies. URLs: sciencedirect.com/article/abs/pii/S0017931019351877 ; sciencedirect.com/article/abs/pii/S0016236117311134 ; boiler fouling review (2012) PDF: scholarlinkinstitute.org/jeteas/articles/Effect%20of%20Fouling%20on%20Heat%20Transfer%20Surfaces%20in%20Boilers.pdf . ([ScienceDirect][14], [scholarlinkinstitute.org][15])

### Option B — **Sealed bypass leg with pressure-balanced flap** (preferred for predictability)

Create a **parallel sealed duct** from a mid-spiral tap to the flue riser. Add a **counterweighted/spring-loaded flap** that cracks open at a designed ∆p (e.g., 100–200 Pa), analogous to internal draft control but entirely within the gas-side network. This gives a **sharp opening threshold** and is **insensitive to fouling** compared with porous media. You can substantiate the furnace’s stability under UL 727 test conditions (steady combustion and draft). Reference to the federal incorporation of UL 727 test conditions: 10 CFR 431.76. URL: law\.resource.org/pub/us/cfr/ibr/006/ul.727.1994.pdf . ([Law Resource][7])

---

## Quick screening workflow (numbers you can plug)

1. **Draft & fan curve.** Establish allowable system ∆p and target overflow **crack-open ∆p** (e.g., 150 Pa).
   *Why:* Curved/spiral passes raise resistance as fouling accumulates. (Curvature→Nu↑ and resistance↑). URL: sciencedirect.com/article/pii/S2214157X23011607. ([ScienceDirect][1])
2. **Pick device type.**
   • If **perforated plate:** choose **β**; compute $K_\mathrm{plate}$ from Idelchik; solve $\Delta p = \tfrac{1}{2}\rho V^2 K$ for required **panel area** A at your expected velocity window. URLs: aft.com/…/perforated-plates-pressure-drop ; sciencedirect.com/article/pii/S0017931024003211 . ([AFT][9], [ScienceDirect][10])
   • If **coarse screen:** use **Armour & Cannon** with ε, wire dia, and layer count to hit the same ∆p at design V. URL: aiche.onlinelibrary.wiley.com/doi/abs/10.1002/aic.690140315 . ([AIChE Journal][12])
3. **Thermal guardrails.** Ensure the **coldest metal** near the overflow remains **≥ (acid dew-point + 10–20 K)**, or confine sub-dew regions to **S-TEN / 316L** class materials and provide drainage/washdown provisions. URLs: watertechnologies.com/handbook/chapter-22-cold-end-deposition-and-corrosion-control ; nipponsteel.com/product/catalog\_download/pdf/A012en.pdf . ([Water Technologies][4], [Perma Pure][5])
4. **Access & cleaning.** Add **full-diameter cleanouts** and keep the panel **rod-cleanable**. Use the spiral’s **single-channel velocity** to your advantage (self-scouring) and avoid dead zones. Vendor guidance on single-channel, self-cleaning behavior: Alfa Laval spiral HX pages. URLs: alfalaval.us/…/welded-spiral-heat-exchangers/unique-features/ ; …/welded-spiral-heat-exchangers/ . ([alfalaval.us][16])

---

## Concrete design targets (starting values to iterate)

* **Porous device choice:** perforated plate with **β = 0.30 ± 0.05**, **hole d = 3–6 mm**, **t/d ≤ 1** (limits jet-contraction losses). Expect **K ≈ 3–6**; refine from Idelchik-based calculators. URL: aft.com/…/perforated-plates-pressure-drop ; arxiv.org/pdf/2304.11730 . ([AFT][9], [arXiv][11])
* **“Coarse” screen alternative:** single layer, **ε ≥ 0.7**, wire **0.4–0.6 mm**, panel area sized so **∆p \~100–200 Pa** at **V ≈ 12–18 m·s⁻¹** flue-gas. Use **Armour & Cannon** to back-solve area. URL: aiche.onlinelibrary.wiley.com/doi/abs/10.1002/aic.690140315 . ([AIChE Journal][12])
* **Cold-end wall temperature:** maintain **≥ 130–160 °C** unless acid-resistant alloy is used (exact number from your SO₃/H₂O—see dew-point charts/calculator). URLs: watertechnologies.com/…/cold-end-deposition-and-corrosion-control ; permapure.com/…/sulfuric-acid-dew-point-calculator/ . ([Water Technologies][4], [Perma Pure][5])
* **Inspection points:** place **thermocouples** at the overflow panel and just downstream; confirm you’re not driving the metal sub-dew during overflow events.

---

## When to prefer a **mechanical bypass flap** instead

If your analysis shows **∆p swings are episodic** (start-up, rare fouling bursts), a **sealed bypass with a spring-weighted flap** gives: (i) **zero** normal leakage, (ii) **sharp** relief threshold, (iii) **no soot blinding**. You still retain the spiral’s heat-transfer advantages in normal operation. Test/labeling still follows **UL 727** appliance conditions. URL: law\.resource.org/pub/us/cfr/ibr/006/ul.727.1994.pdf. ([Law Resource][7])

---

### Bottom line

* Your clarified idea **can work**—**if** the porous element vents **gas→gas only** inside a sealed flue-gas network **and** is sized from established **perforated-plate/screen** correlations, with adequate **cleaning access** and **dew-point margin**. For reliability, **perforated plates or a mechanical bypass flap** usually beat “micro” meshes in sooty oil service.
  References with methods and data: Idelchik-based plate K-factors (aft.com/support/product-tips/perforated-plates-pressure-drop) ([AFT][9]); screen pressure-drop correlations (AIChE J. 1968; sciencedirect.com/article/abs/pii/S0017931005001638) ([AIChE Journal][12], [ScienceDirect][13]); spiral HX fouling/“self-clean” behavior (alfalaval.us …/welded-spiral-heat-exchangers/) ([alfalaval.us][2]); acid-dew-point constraints (watertechnologies.com/…/chapter-22-cold-end-deposition-and-corrosion-control ; permapure.com/…/sulfuric-acid-dew-point-calculator/) ([Water Technologies][4], [Perma Pure][5]); UL 727/NFPA 31 compliance (shopulstandards.com/ProductDetail.aspx?UniqueKey=33922 ; nfpa.org/codes-and-standards/nfpa-31-standard-development/31). ([UL Standards Shop][6], [NFPA][8])

If you want, I can plug **your furnace’s firing rate, stack temp, target draft, and fuel sulfur** into a quick sizing worksheet to pick β, d, and panel area for a perforated overflow—and compare it against a spring-loaded bypass flap on the same ∆p setpoint.

[1]: https://www.sciencedirect.com/science/article/pii/S2214157X23011607?utm_source=chatgpt.com "Dean number only can not determine characteristics of ..."
[2]: https://www.alfalaval.us/products/heat-transfer/plate-heat-exchangers/welded-spiral-heat-exchangers/?utm_source=chatgpt.com "Industrial welded spiral heat exchangers"
[3]: https://www.alfalaval.com/products/heat-transfer/plate-heat-exchangers/welded-spiral-heat-exchangers/technical-information/?utm_source=chatgpt.com "Technical information about Welded spiral heat exchangers"
[4]: https://www.watertechnologies.com/handbook/chapter-22-cold-end-deposition-and-corrosion-control?utm_source=chatgpt.com "Chapter 22 - Cold-end Deposition and Corrosion Control"
[5]: https://www.permapure.com/2021/04/09/sulfuric-acid-dew-point-calculator/?utm_source=chatgpt.com "Sulfuric Acid Dew Point Calculator"
[6]: https://www.shopulstandards.com/ProductDetail.aspx?UniqueKey=33922&utm_source=chatgpt.com "UL 727 - Oil-Fired Central Furnaces - UL Standards"
[7]: https://law.resource.org/pub/us/cfr/ibr/006/ul.727.1994.pdf?utm_source=chatgpt.com "UL 727: Oil-Fired Central Furnaces"
[8]: https://www.nfpa.org/codes-and-standards/nfpa-31-standard-development/31?utm_source=chatgpt.com "NFPA 31 Standard Development"
[9]: https://www.aft.com/support/product-tips/perforated-plates-pressure-drop?utm_source=chatgpt.com "Calculate Pressure Drop Across Sharp-Edged Perforated ..."
[10]: https://www.sciencedirect.com/science/article/pii/S0017931024003211?utm_source=chatgpt.com "A pressure-loss model for flow-through round-hole ..."
[11]: https://arxiv.org/pdf/2304.11730?utm_source=chatgpt.com "A fluid flow model for the pressure loss through perforated ..."
[12]: https://aiche.onlinelibrary.wiley.com/doi/abs/10.1002/aic.690140315?utm_source=chatgpt.com "Fluid flow through woven screens - 1968 - AIChE Journal"
[13]: https://www.sciencedirect.com/science/article/abs/pii/S0017931005001638?utm_source=chatgpt.com "Measurement and correlation of hydraulic resistance ..."
[14]: https://www.sciencedirect.com/science/article/abs/pii/S0017931019351877?utm_source=chatgpt.com "Fouling potential prediction and multi-objective ..."
[15]: https://www.scholarlinkinstitute.org/jeteas/articles/Effect%20of%20Fouling%20on%20Heat%20Transfer%20Surfaces%20in%20Boilers.pdf?utm_source=chatgpt.com "Effect of Fouling on Heat Transfer Surfaces in Boilers"
[16]: https://www.alfalaval.us/products/heat-transfer/plate-heat-exchangers/welded-spiral-heat-exchangers/unique-features/?utm_source=chatgpt.com "Unique features for solving spiral problems"
