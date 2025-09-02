## OpenAI ChatGPT v1.2025.232 

**Q: If a hole were to penetrate earth's surface, passing through the core to the surface at the other side, what would be the effect?**

**A:** Short answer: in an ideal, evacuated, frictionless straight tunnel through Earth’s center, you’d execute simple-harmonic motion. You’d fall to mid-Earth, reach \~7.9 km/s, decelerate symmetrically, and arrive at the far surface momentarily at rest \~42.2 min after you jumped; you’d then oscillate back and forth forever. In the real Earth, rotation, air, and friction ruin this: you’d be deflected into the tunnel wall by Coriolis forces unless the tunnel were aligned with the spin axis; with air and wall losses you’d never reach the far side and would damp out at the center. The tunnel itself is impossible to build because of core pressures and temperatures. ([Wikipedia][1], [Federal Aviation Administration][2], [ADS][3], [National Geographic Education][4], [Science News Explores][5])

Key facts and numbers

* Idealized vacuum + no friction + uniform-density Earth → SHO. One-way travel time ≈ 42.2 min (full period ≈ 84.4 min), with maximum speed at the center $v_{\max}\approx\sqrt{gR}\approx7.91\text{ km/s}$. ([Wikipedia][1], [Federal Aviation Administration][2])
* Realistic density profile (PREM): the one-way fall time along the diameter drops to ≈ 38 min (not exactly SHO). ([Wikipedia][1], [arXiv][6])
* Earth’s rotation: unless the tunnel is exactly pole-to-pole, angular-momentum conservation makes you overtake the surrounding rock as radius decreases, so you slam into the east wall; a 2020 AJP analysis gives conditions for any straight tunnel to be traversable in a rotating body. ([ADS][3])
* Air and friction: with realistic aerodynamic drag and rail/wall losses, the traveler never reaches the far side; the motion damps and settles near the center. Modeling with air shows timescales of many months just to reach mid-Earth without full vacuum. ([Science News Explores][5])
* Engineering reality: inner-core boundary pressures ≈ 330–360 GPa and temperatures ≈ 5200–5700 K make any such borehole structurally impossible with present materials. ([National Geographic Education][7], [PMC][8])

What you’d “feel,” case by case

1. Ideal vacuum, frictionless, non-rotating Earth: weightless after the first instant; peak speed at the center; zero speed at the far surface; perpetual oscillation. ([Wikipedia][1])
2. Add Earth’s rotation: strong sideways deflection from Coriolis in any non-axial tunnel, causing wall impact unless the vehicle is actively guided; a pole-to-pole tunnel removes most of this term. ([ADS][3], [National Geographic Education][4])
3. Add air: rapidly increasing air density and compressive heating create enormous drag; motion damps out and ends at the center unless the tube is near-perfectly evacuated and low-friction. ([Science News Explores][5])
4. Real Earth materials: the lithosphere–mantle–core conditions preclude an open tunnel; rock would flow and close the hole. ([USGS Publications Warehouse][9])

References with URLs

* Gravity train overview and 42.2 min/38 min results: “Gravity train,” en.wikipedia.org/wiki/Gravity\_train. ([Wikipedia][1])
* Klotz, A.R. “The gravity tunnel in a non-uniform Earth,” AJP 83(3), 2015. arxiv.org/abs/1308.1342; ui.adsabs.harvard.edu/abs/2015AmJPh..83..231K/abstract. ([arXiv][6], [ADS][10])
* Orbital/center speed scale $v\approx7.9$ km/s: FAA “Understanding Orbits,” faa.gov/sites/faa.gov/files/about/office\_org/headquarters\_offices/avs/III.4.1.3\_Understanding\_Orbits.pdf. ([Federal Aviation Administration][2])
* Rotation/Coriolis constraint for straight tunnels: Simonič, A., “A note on a straight gravity tunnel through a rotating body,” AJP 88(6), 2020. pubs.aip.org/aapt/ajp/article/88/6/499/1043932. ([AIP Publishing][11])
* Coriolis effect explainer: National Geographic Education, education.nationalgeographic.org/resource/coriolis-effect/. ([National Geographic Education][4])
* Drag and friction damping to the center: Science News Explores, “Falling through Earth might be a long and fruitless trip,” snexplores.org/article/falling-through-earth-might-be-long-and-fruitless-trip. ([Science News Explores][5])
* Earth interior parameters (pressure, temperature): National Geographic Education “Core,” education.nationalgeographic.org/resource/core/; Chen et al., PNAS PMC article on inner-core pressures, pmc.ncbi.nlm.nih.gov/articles/PMC4273394/. ([National Geographic Education][7], [PMC][8])

If you want, I can derive the 42.2 min and $v_{\max}=\sqrt{gR}$ results step-by-step from $a(r)=-(g/R)\,r$ and show where the 38 min PREM correction comes from.

[1]: https://en.wikipedia.org/wiki/Gravity_train "Gravity train - Wikipedia"
[2]: https://www.faa.gov/sites/faa.gov/files/about/office_org/headquarters_offices/avs/III.4.1.3_Understanding_Orbits.pdf?utm_source=chatgpt.com "Understanding Orbits"
[3]: https://ui.adsabs.harvard.edu/abs/2020AmJPh..88..499S/abstract?utm_source=chatgpt.com "A note on a straight gravity tunnel through a rotating body"
[4]: https://education.nationalgeographic.org/resource/coriolis-effect/?utm_source=chatgpt.com "The Coriolis Effect: Earth's Rotation and Its Effect on Weather"
[5]: https://www.snexplores.org/article/falling-through-earth-might-be-long-and-fruitless-trip "Falling through Earth might be a long and fruitless trip"
[6]: https://arxiv.org/abs/1308.1342?utm_source=chatgpt.com "The Gravity Tunnel in a Non-Uniform Earth"
[7]: https://education.nationalgeographic.org/resource/core/?utm_source=chatgpt.com "Core - National Geographic Education"
[8]: https://pmc.ncbi.nlm.nih.gov/articles/PMC4273394/?utm_source=chatgpt.com "Hidden carbon in Earth's inner core revealed by shear ..."
[9]: https://pubs.usgs.gov/gip/dynamic/inside.html?utm_source=chatgpt.com "Inside the Earth [This Dynamic Earth, USGS]"
[10]: https://ui.adsabs.harvard.edu/abs/2015AmJPh..83..231K/abstract?utm_source=chatgpt.com "The gravity tunnel in a non-uniform Earth - ADS"
[11]: https://pubs.aip.org/aapt/ajp/article/88/6/499/1043932/A-note-on-a-straight-gravity-tunnel-through-a?utm_source=chatgpt.com "A note on a straight gravity tunnel through a rotating body"

**Q: To what depth would the hole remain unobstructed?**

**A:** Approximate limit: an “unobstructed” vertical void in rock would stop self-supporting within roughly 2–4 km in strong crystalline rock, and within a few hundred meters to \~1–2 km in weaker sedimentary rock. Beyond that, lithostatic stress and time-dependent creep close the void; at great depth the rock can fail violently (spalling/rockburst) or flow. Empirically, the deepest natural self-sustaining voids are \~2.2 km deep caves; mines at \~4 km require heavy support. ([Smith College Science][1], [Cell][2], [WorldAtlas][3], [Wikipedia][4])

Why those depths

* Lithostatic stress grows ≈27 MPa/km for typical crustal density. A circular, air-filled cavity under near-hydrostatic in-situ stress concentrates hoop stress at ≈2 p at the wall. Setting 2 p ≈ UCS (uniaxial compressive strength) gives a ceiling depth $z\approx \text{UCS}/(2\times27\,\text{MPa/km})$. This yields only a few kilometers for even very strong rocks. ([Smith College Science][1], [Wikipedia][5])
* Real rocks are jointed, anisotropic, wet, and warm. Stand-up time shrinks with depth; rockburst risk rises sharply, so unsupported spans become impractical well before the theoretical ceiling. ([Rocscience][6], [ScienceDirect][7])
* Evidence: Veryovkina Cave ≈2.21–2.212 km (natural limit of self-support); Mponeng Mine ≈4 km but only with intense ground support and cooling; deep boreholes like Kola (12.26 km) are narrow, cased, and still suffer time-dependent closure at \~180 °C. ([Wikipedia][8], [National Geographic][9], [Interesting Engineering][10])

Back-of-envelope ceilings from $z\approx \text{UCS}/(54\,\text{MPa/km})$ (optimistic, ignores joints/heat)
UCS ranges from the cited rock-mechanics compilations. ([Geoengineer][11], [ceae.colorado.edu][12])

* Sandstone 20–170 MPa → \~0.37–3.15 km. ([Geoengineer][11])
* Limestone 30–250 MPa → \~0.56–4.63 km. ([Geoengineer][11])
* Shale 5–100 MPa → \~0.09–1.85 km. ([Geoengineer][11])
* Granite 100–250 MPa → \~1.85–4.63 km. ([Geoengineer][11])
* Basalt 200–300 MPa → \~3.70–5.56 km. ([SpringerLink][13])
* Quartzite 150–300 MPa → \~2.78–5.56 km. ([Geoengineer][11])

Implications

* A through-Earth vacuum tunnel cannot remain unobstructed. Even small-diameter holes start to creep-close at several kilometers; large spans fail much shallower. Temperature rises \~25–30 °C/km accelerate weakening and ductile flow. ([Wikipedia][14])

References (text links)

* Smith College geostatic gradient note: [https://www.science.smith.edu/geosciences/petrology/Notes/GeoPressures.pdf](https://www.science.smith.edu/geosciences/petrology/Notes/GeoPressures.pdf) ([Smith College Science][1])
* Kirsch equations overview: [https://en.wikipedia.org/wiki/Kirsch\_equations](https://en.wikipedia.org/wiki/Kirsch_equations) and ScienceDirect topic page: [https://www.sciencedirect.com/topics/engineering/kirsch-equation](https://www.sciencedirect.com/topics/engineering/kirsch-equation) ([Wikipedia][5], [ScienceDirect][15])
* Rock mass stand-up time and classification: [https://www.rocscience.com/assets/resources/learning/hoek/Practical-Rock-Engineering-Chapter-3-Rock-Mass-Classification.pdf](https://www.rocscience.com/assets/resources/learning/hoek/Practical-Rock-Engineering-Chapter-3-Rock-Mass-Classification.pdf) ([Rocscience][6])
* Rockburst reviews: [https://www.sciencedirect.com/science/article/pii/S2467967422000022](https://www.sciencedirect.com/science/article/pii/S2467967422000022) and [https://www.cell.com/heliyon/fulltext/S2405-8440%2824%2902546-5](https://www.cell.com/heliyon/fulltext/S2405-8440%2824%2902546-5) ([ScienceDirect][7], [Cell][2])
* UCS ranges by rock type: [https://www.geoengineer.org/education/laboratory-testing/unconfined-compression-test](https://www.geoengineer.org/education/laboratory-testing/unconfined-compression-test) and [https://ceae.colorado.edu/\~amadei/CVEN5768/PDF/NOTES8.pdf](https://ceae.colorado.edu/~amadei/CVEN5768/PDF/NOTES8.pdf) and basalt values: [https://link.springer.com/article/10.1007/BF01024770](https://link.springer.com/article/10.1007/BF01024770) ([Geoengineer][11], [ceae.colorado.edu][12], [SpringerLink][13])
* Deepest natural cave: [https://www.nationalgeographic.com/adventure/article/flood-escape-deepest-cave-veryovkina-abkhazia](https://www.nationalgeographic.com/adventure/article/flood-escape-deepest-cave-veryovkina-abkhazia) and list overview: [https://www.worldatlas.com/caves/the-deepest-caves-in-the-world.html](https://www.worldatlas.com/caves/the-deepest-caves-in-the-world.html) ([National Geographic][9], [WorldAtlas][3])
* Deepest mine: [https://en.wikipedia.org/wiki/Mponeng\_Gold\_Mine](https://en.wikipedia.org/wiki/Mponeng_Gold_Mine) ([Wikipedia][4])
* Kola Superdeep Borehole: [https://en.wikipedia.org/wiki/Kola\_Superdeep\_Borehole](https://en.wikipedia.org/wiki/Kola_Superdeep_Borehole) and background: [https://interestingengineering.com/lists/facts-the-superdeep-kola-borehole](https://interestingengineering.com/lists/facts-the-superdeep-kola-borehole) ([Wikipedia][16], [Interesting Engineering][10])
* Geothermal gradient: [https://en.wikipedia.org/wiki/Geothermal\_gradient](https://en.wikipedia.org/wiki/Geothermal_gradient) ([Wikipedia][14])

Bottom line: expect self-support only to roughly 1–2 km in most rocks and at best \~2–4 km in the strongest, driest, least-jointed crystalline rock. Deeper requires support; a “hole through Earth” cannot remain unobstructed. ([Smith College Science][1], [Rocscience][6], [ScienceDirect][7])

[1]: https://www.science.smith.edu/geosciences/petrology/Notes/GeoPressures.pdf?utm_source=chatgpt.com "A Crustal Geostatic Gradient"
[2]: https://www.cell.com/heliyon/fulltext/S2405-8440%2824%2902546-5?utm_source=chatgpt.com "Empirical approaches for rock burst prediction"
[3]: https://www.worldatlas.com/caves/the-deepest-caves-in-the-world.html?utm_source=chatgpt.com "The Deepest Caves In The World"
[4]: https://en.wikipedia.org/wiki/Mponeng_Gold_Mine?utm_source=chatgpt.com "Mponeng Gold Mine"
[5]: https://en.wikipedia.org/wiki/Kirsch_equations?utm_source=chatgpt.com "Kirsch equations"
[6]: https://www.rocscience.com/assets/resources/learning/hoek/Practical-Rock-Engineering-Chapter-3-Rock-Mass-Classification.pdf?utm_source=chatgpt.com "Rock mass classification"
[7]: https://www.sciencedirect.com/science/article/pii/S2467967422000022?utm_source=chatgpt.com "Rockburst in underground excavations: A review of ..."
[8]: https://en.wikipedia.org/wiki/Veryovkina_Cave?utm_source=chatgpt.com "Veryovkina Cave"
[9]: https://www.nationalgeographic.com/adventure/article/flood-escape-deepest-cave-veryovkina-abkhazia?utm_source=chatgpt.com "Epic flood sends cavers scrambling for their lives"
[10]: https://interestingengineering.com/lists/facts-the-superdeep-kola-borehole?utm_source=chatgpt.com "7 facts about the Kola Superdeep Borehole"
[11]: https://www.geoengineer.org/education/laboratory-testing/unconfined-compression-test?utm_source=chatgpt.com "Unconfined Compression Test"
[12]: https://ceae.colorado.edu/~amadei/CVEN5768/PDF/NOTES8.pdf?utm_source=chatgpt.com "strength properties of rocks and rock masses"
[13]: https://link.springer.com/article/10.1007/BF01024770?utm_source=chatgpt.com "Limits on strength and deformation properties of jointed ..."
[14]: https://en.wikipedia.org/wiki/Geothermal_gradient?utm_source=chatgpt.com "Geothermal gradient"
[15]: https://www.sciencedirect.com/topics/engineering/kirsch-equation?utm_source=chatgpt.com "Kirsch Equation - an overview"
[16]: https://en.wikipedia.org/wiki/Kola_Superdeep_Borehole?utm_source=chatgpt.com "Kola Superdeep Borehole"
