# Data Spec Builder vs SAP Workbench — Structural Comparison

## Overview

Both are single-file HTML apps in the DataSpecDashBoard repo used by the Eli Lilly SDIA team for Real-World Evidence study planning. The Data Spec Builder generates Strada data specifications for data programmers; the SAP Workbench generates Statistical Analysis Plans for statisticians.

## Architecture

| | **Data Spec Builder** | **SAP Workbench** |
|---|---|---|
| **File** | `data-spec-builder.html` (~1,200 lines) | `sap-workbench_lit_04232026_V2.html` (~5,500 lines) |
| **Framework** | Vanilla JS — no framework, direct DOM manipulation via `innerHTML` | React 18 (CDN + Babel) — component-based with hooks |
| **State management** | Single global `S` object, mutated directly (e.g. `S.overview.objective_primary = value`) | React `useState` hooks with immutable updates (`setValues`, `updateValue`) |
| **Rendering** | Manual: each section has a render function (`rOutcomes()`, `rAlgorithm()`) returning HTML strings; `rMain()` dispatches by `S.activeSection` | Declarative React components (`StudyDesignSection`, `AnalysisSection`, `GenericSection`) rendered via JSX |

## Section & Field Model

| | **Data Spec Builder** | **SAP Workbench** |
|---|---|---|
| **Section definition** | Flat `SECS` object with 10 sections; fields hardcoded in each render function | `SECTIONS_BASE` with nested field configs (type, placeholder, options, groups); `GenericSection` auto-renders fields from config |
| **Sections** | Overview, Study Periods, Inclusion, Exclusion, Outcomes & Treatment, Cohort Algorithm, Feasibility Checks, Variable Definitions, Negative Controls, Considerations | Study Design (with sub-groups), Main Analysis (with panel switching), Subgroup Analysis, Sensitivity, Diagnostics, TFL Plan, Code Lists, Quality Checklist |
| **Field types** | Textareas, text inputs, toggles, custom arm cards — all hand-built | Rich type system: `textarea`, `text`, `select`, `multiselect`, `date_range`, `enrollment_fill`, `index_date_select`, `adjustment_select`, `secondary_outcomes_list`, etc. |

## Section Organization — Side-by-Side

This shows how the same study design concepts are organized differently across the two tools.

### Population / Eligibility

| **Data Spec Builder** | **SAP Workbench (RWE)** | **SAP Workbench (Feasibility)** |
|---|---|---|
| **3. Inclusion Criteria** — standalone section, list of free-text criteria | Nested inside **1. Study Design → Population** sub-group as a single textarea | **1. Population** — standalone section with inclusion, exclusion, and continuous enrollment |
| **4. Exclusion Criteria** — standalone section, list of free-text criteria | Same sub-group as inclusion, single textarea | Same standalone section as above |
| Continuous enrollment is a field inside **2. Study Periods** | Separate `enrollment_fill` widget in Population sub-group | Included in standalone Population section |

**Key difference:** Data Spec Builder treats inclusion and exclusion as two independent top-level sections (each a dynamic list of individual criteria). SAP Workbench nests them together under a "Population" concept — either as a sub-group within Study Design (RWE) or as its own top-level section (Feasibility).

### Study Periods / Dates

| **Data Spec Builder** | **SAP Workbench (RWE)** | **SAP Workbench (Feasibility)** |
|---|---|---|
| **2. Study Periods** — standalone section with 8 fields: study period, index period, baseline comorbidity, baseline weight, baseline T2D, index event, continuous enrollment, follow-up | Split across two sub-groups within **1. Study Design**: **Study Period** (study dates, index dates, baseline window) and **Follow-up** (intercurrent/censoring/competing events, follow-up period) | **2. Study Period** — standalone section with study dates, index dates, baseline window |

**Key difference:** Data Spec Builder has a single flat section for all time-related fields. SAP Workbench separates "when" (study period sub-group) from "what happens during follow-up" (follow-up sub-group with intercurrent events, censoring, competing events). Feasibility mode strips follow-up entirely.

### Outcomes

| **Data Spec Builder** | **SAP Workbench (RWE)** | **SAP Workbench (Feasibility)** |
|---|---|---|
| Part of **5. Outcomes & Treatment** — primary outcome (textarea), secondary outcomes (textarea) | Inside **1. Study Design → Outcome** sub-group — primary outcome name, definition, type (select), plus structured secondary outcomes list with per-outcome type and definition | No outcomes section |

**Key difference:** Data Spec Builder uses simple free-text fields. SAP Workbench has structured outcome definitions with type classification (binary, time-to-event, continuous, count) that drives downstream analysis method suggestions and TFL recommendations.

### Treatment Arms

| **Data Spec Builder** | **SAP Workbench (RWE)** | **SAP Workbench (Feasibility)** |
|---|---|---|
| Part of **5. Outcomes & Treatment** — waterfall arm cards with drug list, logic operator (ANY/ALL/SEQUENTIAL/NONE), min Rx, index date rule, concomitant overlap settings; plus post-index tracking (switching, augmentation, dose escalation) | Inside **1. Study Design → Treatment Arms** sub-group — simple treated/comparator name + definition textareas | **3. Treatment Arms** — standalone section, same simple treated/comparator fields |

**Key difference:** Data Spec Builder has a rich waterfall-based arm definition system (ordered priority, drug logic operators, concomitant/sequential rules, anchor drugs). SAP Workbench uses simple two-arm (treated vs comparator) text definitions. Data Spec Builder also tracks post-index treatment changes (switching, augmentation, dose escalation).

### Index Date

| **Data Spec Builder** | **SAP Workbench (RWE)** | **SAP Workbench (Feasibility)** |
|---|---|---|
| Defined per-arm in the waterfall cards (index date rule: first Rx, combo met, switch date, custom) | **1. Study Design → Index Date** sub-group — separate select+custom for treated and control | **4. Index Date** — standalone section, same select+custom fields |

**Key difference:** Data Spec Builder ties index date logic to each treatment arm in the waterfall. SAP Workbench defines index dates globally for treated and control groups.

### Algorithm / Cohort Construction

| **Data Spec Builder** | **SAP Workbench (RWE)** | **SAP Workbench (Feasibility)** |
|---|---|---|
| **6. Cohort Algorithm** — free-text pseudo-algorithm textarea; AI "Suggest Algorithm" feature | No equivalent section — cohort logic is implicit in the design fields | **5. Attrition Table** — auto-generated from population/treatment/index date fields in STRADA format |

**Key difference:** Data Spec Builder has an explicit step-by-step algorithm section (often AI-assisted). SAP Workbench doesn't have a manual algorithm section; Feasibility mode auto-generates an attrition table instead.

### Analysis Methods

| **Data Spec Builder** | **SAP Workbench (RWE)** |
|---|---|
| No equivalent — analysis is out of scope for a data spec | **2. Main Analysis** — panel-switching between Fixed and Longitudinal analysis types, with estimand, adjustment method, outcome method, effect measure. Plus **3. Subgroup Analysis**, **4. Sensitivity / Unmeasured Confounding**, **5. Diagnostics** |

**Key difference:** This is entirely SAP Workbench territory. Data Spec Builder focuses on "what data to pull"; SAP Workbench focuses on "how to analyze it."

### Sections unique to Data Spec Builder

| Section | Purpose |
|---|---|
| **1. Study Overview** | Objectives (primary/secondary), general notes |
| **7. Feasibility Checks** | Toggle list of feasibility items to verify |
| **8. Variable Definitions** | Structured table of variables with name, definition, type, codelist links |
| **9. Negative Controls** | NCO source file, datasets, notes |
| **10. Considerations** | Missing data, data source notes, output format, other |

### Sections unique to SAP Workbench

| Section | Purpose |
|---|---|
| **2. Main Analysis** | Estimand, confounding adjustment, outcome method, effect measure (fixed vs longitudinal panels) |
| **3. Subgroup Analysis** | Define subgroups with analysis assignments |
| **4. Sensitivity** | Unmeasured confounding methods (E-value, negative controls, etc.) |
| **5. Diagnostics** | Covariate balance, PS overlap, weight distribution, PH assumption checks |
| **6. TFL Plan** | Select tables/figures/listings from a library, auto-suggested based on design choices |
| **7. Code Lists** | Variable-to-codelist mapping table (auto-populated from design fields) |
| **8. Quality Checklist** | Pre-submission quality verification items |

## Study Type Handling

| | **Data Spec Builder** | **SAP Workbench** |
|---|---|---|
| **Types available** | Simple toggle (Comparative / Descriptive) — mostly cosmetic | Deep branching: RWE Comparative, Feasibility, Cost/HCRU, Regulatory |
| **Impact on UI** | Minimal — same sections shown for both types | Each type shows different sections (`FEASIBILITY_SECTIONS` vs filtered `SECTIONS_BASE`); per-type value isolation via `valuesByType` |
| **Type-specific guidance** | None | `HINTS` object provides per-type guidance panels (e.g. Cost/HCRU estimand guidance, design guidance) |

## Persistence

| | **Data Spec Builder** | **SAP Workbench** |
|---|---|---|
| **Auto-save** | Yes — `localStorage` via `saveState()` on every change | No auto-save |
| **Manual save/load** | Not needed (auto-persists) | Save/Load to `.json` file; includes `valuesByType` for per-type isolation (v2 format) |
| **Preference learning** | None | Preference Agent stores writing style observations across sessions via `PREF_STORAGE_KEY` in `window.storage` |

## AI Features

| | **Data Spec Builder** | **SAP Workbench** |
|---|---|---|
| **Scope** | Single "Suggest Cohort Algorithm" button | Multi-step Literature Review → Study Proposal pipeline |
| **Workflow** | One-shot: sends study context to Claude, returns algorithm text | Upload PDFs → generate proposal → chat refinement → accept & pre-fill all workbench sections |
| **Preference learning** | None | `learnFromSession()` observes accepted proposals and user refinements to improve future suggestions |

## Export

| | **Data Spec Builder** | **SAP Workbench** |
|---|---|---|
| **Format** | HTML string → Blob → `.doc` download | Same approach |
| **Outputs** | Single "Export to Word" | Two exports: "Export SAP to Word" and "Export Spec to Word", both with auto-generated TOC |
| **Content** | Objectives, study details, algorithm, feasibility, variables, NCOs | Full SAP: estimand, design, analysis methods, confounding adjustment, diagnostics, TFL plan, sensitivity analyses |

## Key Patterns

- **Data Spec Builder** favors simplicity: global mutable state, string-template rendering, flat section list. Easy to understand and modify.
- **SAP Workbench** favors configurability: React component tree, declarative field configs, deep study-type branching, AI integration. More powerful but significantly more complex.
- Both are zero-build single HTML files that run entirely client-side.
