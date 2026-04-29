# Rapid Response CDS: Sharing Across Institutions — Reference Document
**Version:** 1.0.1  
**Last Updated:** 2026-04-29  
**Conference:** AMIA Amplify 2026, Denver CO, May 18–21

---

## 1. Relevant Organizations

| Organization | Role in CDS Sharing | URL |
|---|---|---|
| **Pediatric CDS Collaborative (PCC)** | Multi-institutional, grant-funded collaborative for pediatric emergency CDS development and sharing; core organizing vehicle for this panel | *(internal — no public URL confirmed)* |
| **HL7 International — CDS Workgroup** | Stewards CDS Hooks, FHIR PlanDefinition profiles, and CPG-on-FHIR IG | https://www.hl7.org/Special/committees/dss/ |
| **SMART Health IT (Boston Children's Hospital)** | Originating partner for CDS Hooks (with Cerner); maintains SMART on FHIR authorization framework; now hosted under HL7 | https://smarthealthit.org |
| **ONC (Office of the National Coordinator for Health IT)** | HTI-1 rule mandates FHIR R4 + SMART + CDS Hooks support for certified EHRs; drives interoperability floor | https://www.healthit.gov |
| **AHRQ — CDS Connect** | Formerly hosted a public CDS artifact repository and authoring tool; **retired April 2025** — cautionary example for platform dependency | https://cds.ahrq.gov *(retired)* |
| **NLM — VSAC** | National repository for value sets used in CDS rules and quality measures; eCQI toolchain anchor | https://vsac.nlm.nih.gov |
| **CDC — CDSi Program** | Publishes immunization schedule logic as structured, machine-readable CDS artifacts (XML/spreadsheet); one of the most mature federal CDS distribution programs | https://www.cdc.gov/vaccines/programs/iis/cdsi.html |
| **AAP CDSi** | AAP's Clinical Decision Support Innovation collaborative; vaccine-specific CDS program | https://www.aap.org/cdsi |
| **MITRE Corporation** | Produces open FHIR-based CDS IGs for public health emergencies (anthrax mass casualty IG; COVID-19 CDS tools) | https://www.mitre.org |
| **cqframework (open-source consortium)** | Maintains the CQL specification, reference compiler, and Clinical Reasoning module for FHIR | https://github.com/cqframework |
| **Epic UserWeb Community Library** | Gated (Epic credentials required); primary mechanism for Epic-to-Epic CDS content sharing | https://userweb.epic.com |
| **KLAS Research** | Independent vendor research; KLAS CDS studies useful for documenting market gaps | https://klasresearch.com |

---

## 2. Relevant Standards

### 2a. CDS Hooks (HL7)

**What it is:** A RESTful, JSON-over-HTTPS specification that allows an EHR ("CDS Client") to invoke external CDS services at defined workflow hook points and render returned guidance cards inline in the clinician's workflow.

**Current version:** CDS Hooks 2.0.1 (STU 2, Release 2) — errata release published via HL7 FHIR IG Publisher; based on FHIR R4. Originating partnership: SMART Health IT + Cerner. HL7 CDS Workgroup now stewards.

**Hook lifecycle:**
1. EHR calls `GET /cds-services` → discovers available hooks and prefetch templates
2. On a workflow event (`patient-view`, `order-select`, `order-sign`, `encounter-start`, `appointment-book`, `encounter-discharge`), EHR POSTs context + FHIR resources (or token) to the CDS Service
3. CDS Service returns **cards**: *information*, *suggestion*, *app link*, or *system action*

**What CDS Hooks cannot do:** Transfer complete order-set definitions between institutions. Cards return guidance; the actual order-set build remains site-specific.

**Reference:** https://cds-hooks.hl7.org

---

### 2b. FHIR PlanDefinition / CPG-on-FHIR

**What it is:** FHIR R4 `PlanDefinition` is the machine-readable representation of a clinical protocol, order set, or guideline. CPG-on-FHIR (Clinical Practice Guidelines on FHIR) is the HL7 Implementation Guide defining how to express computable guidelines using `PlanDefinition`, `ActivityDefinition`, `Library`, and `ValueSet` resources.

**Current status:** CPG-on-FHIR v2.0.0 (current build); IG is mature but real-world EHR execution support remains limited — most production use wraps PlanDefinition logic in a CDS Hooks service or transpiles to vendor-native build.

**Important note:** A published FHIR PlanDefinition is *portable in principle* but requires local build, terminology mapping, and governance to be clinically active in any EHR.

**Reference:** https://hl7.org/fhir/uv/cpg/

---

### 2c. CQL (Clinical Quality Language)

**What it is:** A high-level, human-readable query language for expressing clinical logic, maintained by HL7 and the cqframework consortium. CQL is the primary logic layer for both FHIR-based CDS rules and electronic clinical quality measures (eCQMs).

**Key relationship:** CQL compiles to ELM (Expression Logical Model) — a canonical XML/JSON format that execution engines consume. CDS Hooks services commonly execute CQL-based rules against FHIR data.

**Limitation for emergency CDS:** Few EHRs natively execute CQL at the point of care. Most production deployments wrap CQL in an external CDS Hooks service. Epic executes CQL for eCQM reporting but CQL-at-the-point-of-care requires the CDS Hooks integration path.

**References:**
- https://cql.hl7.org
- https://github.com/cqframework/clinical_quality_language

---

### 2d. VSAC (Value Set Authority Center, NLM)

**What it is:** The authoritative national repository for value sets used in CDS rules, eCQMs, and FHIR IGs. Value sets define the coded concept lists (ICD-10, SNOMED, LOINC, RxNorm, CVX) that logic rules evaluate against.

**Emergency CDS limitation:** VSAC turnaround for new value sets (novel pathogens, supply shortage codes) can take weeks — a significant bottleneck for rapid emergency CDS deployment. Novel conditions or shortage-specific concepts may not yet exist in any standard vocabulary.

**Reference:** https://vsac.nlm.nih.gov

---

### 2e. SMART on FHIR

**What it is:** An OAuth 2.0-based authorization framework for launching apps within an EHR workflow, using standardized FHIR API access. Frequently paired with CDS Hooks (a CDS Hooks card can contain a SMART app link as its action).

**Relevance to CDS sharing:** Enables vendor-neutral app delivery within EHR workflows; a PCC-developed tool deployed as a SMART app could theoretically run across Epic, Oracle Health, and MEDITECH sites without re-building native EHR logic.

**Reference:** https://smarthealthit.org / https://hl7.org/fhir/smart-app-launch/

---

## 3. EHR Vendor Sharing Mechanisms

| EHR Platform | Sharing Mechanism | Mechanism Type | Gated? | Notes |
|---|---|---|---|---|
| **Epic** | Community Library | Customer-to-customer content repository (UserWeb) | ✅ Epic credentials | Search, download, and import build records (BPAs, order sets, toolbars); import still requires local build/testing |
| **Epic** | Turbocharger packages | Vendor-issued content bundles distributed via UserWeb | ✅ Epic credentials | Faster to deploy than community content; no peer review |
| **Epic** | CDS Hooks (Hyperspace) | Standards-based external CDS service invocation | Partial — requires per-tenant setup | Epic is a CDS Hooks client; services must be individually configured |
| **Oracle Health (Cerner)** | Partner Gallery | Certified third-party app catalog | ✅ ORCA credentials | Not a CDS-content repository; oriented toward app vendors |
| **Oracle Health (Cerner)** | CDS Hooks (PowerChart) | Standards-based external CDS service invocation | Partial — per-tenant setup | More permissive API than Epic; FHIR R4 support via OHCI migration |
| **MEDITECH** | MUSE Community (museweb.org) | Independent user community (mailing lists, conf handouts, NPR snippets) | MUSE membership | Education-oriented; no programmatic import; informal text/screenshot sharing |
| **Veradigm (formerly Allscripts)** | App Expo (Veradigm Developer Program) | Certified-app catalog; SDK/API for third-party integrators | Developer enrollment | Not a CDS content-sharing platform; no curated CDS library comparable to Epic |

**Key disparity:** Epic and Oracle Health have richer technical sharing infrastructure. MUSE and Veradigm communities are active for education and informal exchange but lack structured CDS content import mechanisms. This makes standards-based approaches (CDS Hooks + FHIR + CQL) *more* important for MEDITECH and Veradigm sites, not less.

---

## 4. Published Implementations

### 4a. Standards-Based CDS Deployments

| Study / Implementation | Setting | Standards Used | Outcome | Citation |
|---|---|---|---|---|
| ReImagine EHR — neonatal bilirubin SMART app | University of Utah Health | SMART on FHIR + CDS Hooks | Multi-app deployment; SMART on FHIR cited in 500+ hospitals | Kawamoto et al. (2021) — [PMC8325485](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC8325485/) |
| MDCalc reference app via CDS Hooks | Single-center ED, 70 providers | CDS Hooks | CDS Hooks prompts significantly increased SMART app utilization (cluster-RCT) | Powell et al. (2022) — [PMC9382378](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC9382378/) |
| DDInteract — drug-drug interaction CDS | Academic medical center | FHIR R4 + CQL + CDS Hooks + SMART | End-to-end blueprint for standards-based CDS; pilot deployed | Heale et al. (2022), *Int J Med Informatics* — [PMC9703934](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC9703934/) |
| CQL/CDS Hooks/FHIR imaging CDS in Epic | Mass General Brigham (Epic) | CQL + CDS Hooks + FHIR R4 | 12-month retrospective; evaluated accuracy and false-positive rates for imaging appropriateness CDS | Isabelle et al. (2025), *JAMIA Open* — [PMC12309839](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC12309839/) |
| CDS Hooks adoption scoping review | Literature review (44 articles) | FHIR + SMART + CQL + CDS Hooks | Only 5/44 (11%) reached real-world deployment; only 2 had RCT components | Dziuban et al. (2021), *AMIA Annu Symp* — [PMC8416232](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC8416232/) |
| CPG-on-FHIR + GRADE-based evidence | Academic (CEOsys project) | CPG-on-FHIR + EBM-on-FHIR | Proof-of-concept linking systematic review evidence to computable guideline | Lichtner et al. (2023), *J Biomed Informatics* |

### 4b. Emergency / Rapid-Response CDS Deployments

| Event / Use Case | Institution(s) | CDS Mechanism | Notes |
|---|---|---|---|
| **IV fluid shortage — Hurricane Helene** (Sept 2024) | PCC member institutions | EHR-native order set sharing via PCC | Rapid cross-institutional order set sharing for Baxter IV fluid conservation protocols; concrete PCC emergency activation case |
| **Measles outbreak response** | PCC member institutions | EHR-native screening workflows, order sets | Exposure screening, test ordering, public health notification, isolation protocols; PCC facilitated sharing of build documentation across sites |
| **Ebola / Marburg preparedness** | PCC member institutions | EHR-native screening BPAs + order sets | Proactive development and shelf-readiness; not reactive |
| **Avian influenza preparedness** | PCC member institutions | EHR-native; proactive build | Example of preparedness-before-crisis model |
| **CDC CDSi — immunization schedule logic** | Nationwide (IIS/EHR integrations) | Machine-readable XML/spreadsheet CDS artifacts | Most mature federal CDS distribution program; vaccine-specific |
| **MITRE anthrax mass-casualty IG** | Federal/public health | FHIR-based CDS IG | Demonstrates standards-based emergency CDS representation | 
| **AHRQ CDS Connect** | Nationwide (repository) | Web-based CDS artifact authoring/repository | **Retired April 2025** — cautionary example: centralized repository dependency without sustainable funding model |

---

## 5. Known Limitations

These are the structural barriers that motivate the "proposed enhancements" section of the panel.

### Vendor-native sharing (Epic/Oracle Health)
- **Credential-gated access:** Epic Community Library is inaccessible to non-Epic sites and to the public — precludes cross-vendor sharing
- **Logic ≠ build:** Even within Epic, a downloaded Community Library item requires local configuration, terminology mapping, and governance review before clinical activation
- **No emergency fast-lane:** No formalized expedited approval pathway for Community Library content during a declared public health emergency
- **Turbocharger is vendor-issued:** Content is vendor-controlled, not community-curated; lag between clinical need and availability

### Standards-based sharing (CDS Hooks / FHIR / CQL)
- **Per-tenant onboarding required:** CDS Hooks services must be individually configured per institution and per EHR environment — no national directory of trusted, subscribable CDS services
- **CQL execution gaps:** Most EHRs do not natively execute CQL at the point of care; production deployments require an external CDS Hooks service layer
- **Order-set portability not solved:** CDS Hooks cards return information/suggestions, not importable order-set definitions — the "transfer" problem remains
- **VSAC turnaround bottleneck:** Novel pathogen or shortage-specific value sets can take weeks — unacceptable for emergency response timelines
- **Latency constraints:** CDS Hooks best-practices recommend ≤500 ms response budgets; network, FHIR API performance, and rate limits affect real-world reliability
- **Adoption gap:** Dziuban et al. found only 11% of published CDS Hooks studies had reached real-world deployment as of 2021

### Repository sustainability
- **AHRQ CDS Connect retirement (April 2025)** is the definitive cautionary example — a well-designed national repository did not survive a funding transition; artifacts and community infrastructure were lost
- Many academic GitHub repositories are abandoned within 2–3 years of publication

### Pediatric-specific
- Pediatric concepts are systematically underrepresented in standard vocabularies (SNOMED, LOINC, RxNorm) — weight-based dosing, age-normalized ranges, and neonatal/adolescent edge cases require local extensions
- No pediatric-specific emergency CDS sharing framework exists at the federal or standards level; PCC fills this gap informally

---

## 6. References

### Standards and Specifications
- **CDS Hooks 2.0.1 (STU 2)** — HL7: https://cds-hooks.hl7.org
- **CPG-on-FHIR Implementation Guide (v2.0.0)** — HL7: https://hl7.org/fhir/uv/cpg/
- **CQL Specification** — HL7/cqframework: https://cql.hl7.org
- **SMART App Launch Framework** — HL7: https://hl7.org/fhir/smart-app-launch/
- **VSAC** — NLM: https://vsac.nlm.nih.gov
- **FHIR R4** — HL7: https://hl7.org/fhir/R4/

### Peer-Reviewed Literature
- Kawamoto et al. (2021) — ReImagine EHR, SMART on FHIR multi-app deployment: [PMC8325485](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC8325485/)
- Powell et al. (2022) — CDS Hooks + SMART app utilization cluster-RCT: [PMC9382378](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC9382378/)
- Heale et al. (2022) — DDInteract (FHIR + CQL + CDS Hooks + SMART end-to-end), *Int J Med Informatics*: [PMC9703934](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC9703934/)
- Isabelle et al. (2025) — CQL/CDS Hooks/FHIR imaging CDS in Epic (MGH), *JAMIA Open*: [PMC12309839](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC12309839/)
- Dziuban et al. (2021) — CDS Hooks adoption scoping review, *AMIA Annu Symp*: [PMC8416232](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC8416232/)
- Lichtner et al. (2023) — CPG-on-FHIR + GRADE evidence (CEOsys), *J Biomed Informatics*

### Open-Source Repositories
- cqframework / clinical_quality_language: https://github.com/cqframework/clinical_quality_language
- HL7 / cqf-recommendations (CPG-on-FHIR examples incl. opioid MME IG): https://github.com/cqframework/opioid-cds
- CEOsys / cpg-on-ebm-on-fhir: https://github.com/CEOsys/cpg-on-ebm-on-fhir

### Policy and Program References
- ONC HTI-1 Final Rule (FHIR R4 + CDS Hooks certification requirements): https://www.healthit.gov/topic/laws-regulation-and-policy/health-data-technology-and-interoperability
- AHRQ CDS Connect (retired April 2025): https://cds.ahrq.gov
- CDC CDSi Program: https://www.cdc.gov/vaccines/programs/iis/cdsi.html

---

*Document compiled from prior research conversations and panel development sessions. Verify PMC links before distribution. Do not distribute Epic UserWeb URLs externally.*
