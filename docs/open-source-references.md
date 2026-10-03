# Open-source references

Checked on 3 October 2026. This project will build an independent traffic simulation engine for Indian urban conditions. Existing simulators provide useful models, comparisons and interface ideas. They do not establish that our demand assumptions, driver behaviour or enforcement effects are accurate for Bengaluru, Mumbai or Hyderabad.

No upstream source code has been copied into this scaffold. The project’s own software license remains **unresolved**; the licenses below apply to the referenced projects.

## SUMO: network preparation and vehicle interactions

[Eclipse SUMO](https://github.com/eclipse-sumo/sumo) is actively maintained. Its official download page lists **1.27.1, released 25 June 2026**, and describes continuing development and nightly builds. This is current software, even though the project and many underlying models are much older. [Release evidence](https://eclipse.dev/sumo/docs/Downloads.html).

The repository [LICENSE](https://github.com/eclipse-sumo/sumo/blob/main/LICENSE) is **EPL-2.0**. Its [NOTICE](https://github.com/eclipse-sumo/sumo/blob/main/NOTICE.md) declares **GPL-2.0-or-later** as a secondary option under the conditions specified by EPL-2.0. Third-party components have their own terms; a repository-level license does not cover every bundled component identically. [Component licenses](https://sumo.dlr.de/docs/Libraries_Licenses.html).

Study the [OSM Web Wizard](https://sumo.dlr.de/docs/Tutorials/OSMWebWizard.html) for the workflow from a selected map region to a road network, vehicle mix and simulation. Its through-traffic parameter addresses journeys crossing the region boundary. Its public-transport import uses synthetic schedules, and its default trip generation is random. Those are useful prototype assumptions, not observed daily travel demand.

Study the [sublane model](https://sumo.dlr.de/docs/Simulation/SublaneModel.html) for vehicles with different widths, motorcycles sharing lane space, lateral encroachment and gap acceptance. Our design should explicitly choose its spatial resolution and explain the computational cost. These capabilities do not by themselves establish a calibrated model of wrong-way driving, junction blocking or the effect of a police officer.

SUMO remains a reference and possible future comparison tool. The application must run without SUMO binaries, TraCI, libsumo or SUMO helper libraries. Do not port its implementation into our engine under an assumed permissive license.

## A/B Street: inspectable scenarios and planning interfaces

[A/B Street](https://github.com/a-b-street/abstreet) uses **Apache-2.0**, verified in its [LICENSE](https://github.com/a-b-street/abstreet/blob/main/LICENSE). Its latest published simulator release is [v0.3.49, 9 January 2024](https://github.com/a-b-street/abstreet/releases/tag/v0.3.49). The main repository also has a [10 September 2025 commit](https://github.com/a-b-street/abstreet/commit/0964f29315820c91b171b585eb51e300164e9197). The README’s 2025 update explains that development has expanded into several related planning tools. These dates support continued activity and a change of focus, not a claim of frequent new simulator releases.

Study how it makes street and intersection edits understandable and lets users inspect traffic and multimodal journeys. Its [traffic simulation documentation](https://a-b-street.github.io/docs/tech/trafficsim/index.html) points to the [sim crate](https://github.com/a-b-street/abstreet/tree/main/sim/src), including separate scheduling, transit, trips and analytics components. This separation is useful when designing our own engine and UI contracts.

Do not assume that importing an Indian map makes its behavioural assumptions appropriate for Indian mixed traffic. Do not treat its bundled map data, fonts, icons or other assets as covered solely by the software license; its [user guide](https://a-b-street.github.io/docs/user/index.html) lists separate data and asset sources.

## OMoSim: population and activity-based demand

[OMoSim](https://github.com/L-Strobel/omosim), formerly OMOD, is a mobility-demand generator. Its **MIT license** is verified in the [LICENSE](https://github.com/L-Strobel/omosim/blob/master/LICENSE). Recent maintenance includes [v2.6.3, 24 July 2026](https://github.com/L-Strobel/omosim/releases/tag/v2.6.3), and a [22 September 2026 commit](https://github.com/L-Strobel/omosim/commit/d4668aa8acc65f5daa44bd076f832fa1e6a441fa).

Study its daily activity diaries, building and land-use inputs, population definition, mode choice and calibration workflow. Its output suggests an interface in which a demand model supplies people, activities and journeys to a separate traffic engine. This is particularly useful for working hours, population changes and travel-mode scenarios. The recent release also exposes transit legs and separates road-network input from more recent point-of-interest data. [README and calibration documentation](https://github.com/L-Strobel/omosim/blob/master/README.md), [release changes](https://github.com/L-Strobel/omosim/releases/tag/v2.6.3).

The authors explicitly state that calibration used the German national household travel survey and that performance outside Germany is uncertain. Neither building footprints nor the ability to run worldwide validates Indian commute patterns. Local household, workplace and traffic-count evidence will be needed before presenting generated demand as representative.

## Historical Indian precedent

IIT Bombay’s [SiMTraM](https://www.civil.iitb.ac.in/tvm/SiMTraM_Web/html/index.html) explored heterogeneous, non-lane-based traffic. Its [SourceForge distribution](https://sourceforge.net/projects/simtram/) lists **GPLv3**, with the last update **29 April 2013** and latest downloadable version **0.1.1**. It is useful historical context for mixed-traffic research. We found no recent public maintenance and will not use it as a current dependency or copy its code.

## Implementation and provenance policy

Implement our own engine from documented mechanisms and explicitly stated assumptions. Record the source of a borrowed idea beside the relevant design decision. A reference citation does not replace a license grant for copied code.

Any future code reuse must identify the exact upstream revision, file-level license, notices and modifications before entering the repository. Preserve required copyright and license notices, and applicable Apache notices. Keep copied material distinguishable from independently authored code. Missing or unclear permission means no copying.

Track map and dataset permissions separately from software permissions. OpenStreetMap data uses **ODbL**, with attribution and database obligations described on its [copyright page](https://www.openstreetmap.org/copyright). Record extract provenance and display required attribution when importing data. Model accuracy and an open-data license are separate questions.
