# Explainable Procurement Fraud Investigation using Knowledge Graphs and Multi-Layer Risk Analytics

Developed as part of the course on "Forensic Analytics" at Vellore Institute of Technology, Chennai.

by Nesetha J C, Hariharan M

**Subject Code:** CSE4074  
**Semester:** Fall 2026-27  
**Professor:** Dr. S. Johanan Joysingh

---

## Project Overview

This project develops an explainable procurement-investigation framework that combines forensic screening methods, relationship-aware knowledge graph analysis, evidence convergence, and a weighted investigation-priority score. It is designed to identify procurement records that warrant further examination, not to declare that a transaction is fraudulent.

The project uses California Statewide Procurement and Contracting System (SCPRS) purchase-order data from 2012–2015 as the real procurement backbone. Because the source data does not contain complete employee, invoice, and payment relationships, controlled synthetic layers are created for those entities. Six controlled scenarios are injected to test whether the corresponding forensic rules can identify their intended patterns.

> **Important interpretation:** An anomaly is not proof of fraud. The risk score represents investigation priority, not a probability of fraud or a legal finding. The controlled-case detection result is not real-world fraud-detection accuracy.

## Objectives

- Prepare and validate procurement data for forensic analysis.
- Create stable identifiers and a canonical purchase-order model.
- Add controlled synthetic employee, invoice, and payment relationships where source relationships are unavailable.
- Apply complementary forensic screening methods and represent entities and relationships in a knowledge graph.
- Combine risk indicators into a transparent, weighted investigation-priority score.
- Reconstruct selected cases and evaluate the rules against known controlled scenarios.

## Dataset and preparation

- **Source:** California Statewide Procurement and Contracting System (SCPRS) Purchase Order Data, 2012–2015.
- **Original data:** 344,504 records and 32 columns.
- **Analytical line items:** 318,962 after filtering missing or invalid purchase dates; 25,542 records were removed by this date-validity filter.
- **Unique purchase orders:** 187,080.
- **Departments:** 108.
- **Suppliers in the analytical subset:** 24,260.

The notebook cleans date and numeric fields, checks data quality, resolves entities, constructs a PO-level baseline, and builds synthetic employee/invoice/payment layers. Missing monetary values are not treated as zero. The source dataset itself is not included in this repository; users must obtain it from its authorized source and comply with its terms of use.

## Methodology

1. **Dataset loading and profiling** — inspect columns, data types, missing values, date range, and procurement statistics.
2. **Cleaning and preprocessing** — normalize text and supplier codes, convert dates and monetary fields, and retain records with valid purchase dates in the study period.
3. **Entity resolution and baseline construction** — assign identifiers to line items, purchase orders, departments, and suppliers; validate mappings and build a canonical PO table.
4. **Synthetic relationship layers** — assign POs to 350 synthetic employees across 108 departments; create invoice and payment records with controlled timing and amount relationships.
5. **Controlled scenario injection** — create five known cases for each of six scenario types, with a separate ground-truth registry.
6. **Forensic analytics** — run Benford's Law screening, duplicate-invoice detection, statistical outlier analysis, employee–vendor concentration analysis, temporal anomaly analysis, PO–invoice price variance analysis, split-purchase analysis, and round-number analysis.
7. **Knowledge graph and network analysis** — construct a directed graph of Employee, Supplier, Purchase Order, Invoice, and Payment entities and their relationships; attach forensic attributes to relevant nodes.
8. **Evidence convergence and risk scoring** — combine indicators using predefined weights, categorize investigation priority, and reconstruct selected cases.
9. **Controlled evaluation and visualization** — compare the primary rule signals against the 30 controlled ground-truth cases and visualize the findings.

## Knowledge graph

The graph models five entity types: **Employee, Supplier, Purchase Order, Invoice, and Payment**. Its main directed relationships are:

- Employee → Purchase Order (`CREATED`)
- Purchase Order → Supplier (`SUPPLIED_BY`)
- Purchase Order → Invoice (`HAS_INVOICE`)
- Invoice → Payment (`PAID_BY`)
- Employee → Supplier (`INTERACTS_WITH`)

The recorded graph contains **585,855 nodes and 836,213 edges**. It is used for traceability and relationship context; the project does not use a graph neural network to predict fraud.

## Forensic methods and risk weights

| Risk component | Weight |
|---|---:|
| PO–invoice price variance | 25 |
| Duplicate invoice | 20 |
| Employee–vendor concentration | 20 |
| Statistical outlier | 10 |
| Temporal anomaly | 10 |
| Split purchase | 10 |
| Round-number signal | 5 |
| **Maximum theoretical score** | **100** |

The weights are transparent, predefined heuristic values; they are not learned probabilities. The score is the sum of applicable component weights, bounded between 0 and 100. Categories are Low (0–19), Moderate (20–39), High (40–59), and Critical (60–100).

## Key results

- The PO-level risk assessment covers **187,080 purchase orders**.
- Observed investigation-risk scores range from **0 to 60**; the maximum observed score is 60, even though the theoretical maximum is 100.
- Overall risk categories: **127,408 Low (68.103%)**, **58,246 Moderate (31.134%)**, **1,425 High (0.762%)**, and **1 Critical (0.001%)**.
- The highest-priority case is **PO_051985**, score **60 (Critical)**, with duplicate, employee–vendor concentration, temporal, and split-purchase indicators. This is a case-prioritization result, not a confirmed fraud finding.
- **PO_008062** is a high-priority example with score **55 (High)** and statistical-outlier, duplicate-invoice, round-number, and employee–vendor-concentration indicators.
- All **30 deliberately injected controlled cases** were detected through their intended primary forensic signals (30/30, or 100% controlled-case detection under this experiment). This does not establish real-world accuracy, precision, recall, or false-positive rate.

The broad population flags are screening candidates. For example, statistical outliers or concentrated employee–supplier relationships can arise for legitimate reasons and require contextual verification.

## Repository contents

- `Forensic_Project.ipynb` — exported Google Colab notebook containing the project workflow, analysis, case reconstruction, and visualizations.
- `PROJECT_DOCUMENTATION.md` — standalone project explanation structured to parallel the review presentation.
- `requirements.txt` — Python package dependencies for the main analytical libraries.

## How to run

1. Open `Forensic_Project.ipynb` in Google Colab.
2. Obtain the authorized SCPRS source CSV for the 2012–2015 study period.
3. Run the notebook cells in order.
4. When prompted by the notebook, upload the source CSV file.
5. Review the generated data-quality summaries, forensic indicators, graph, risk scores, controlled-case evaluation, and visualizations.
6. The final notebook cell exports analytical CSV tables into a ZIP file for download.

The notebook was authored for Google Colab and uses `google.colab.files` for uploading the source CSV and downloading exported results. A local Jupyter environment would require adapting those upload/download calls.

## Limitations and responsible interpretation

- Employee, invoice, payment, and injected scenario data are controlled synthetic additions, not original verified SCPRS relationships.
- Controlled scenarios are designed test cases and should not be presented as real fraud cases found in the source procurement data.
- The scoring weights are heuristic and require expert validation and sensitivity analysis before operational use.
- Indicators such as Benford deviation, high transaction value, duplicate patterns, round numbers, and employee–vendor concentration can have legitimate explanations.
- The framework prioritizes records for document review; it does not establish intent, collusion, loss, or legal wrongdoing.

## Team

- Nesetha J C
- Hariharan M

## Course details

- Course: Forensic Analytics
- Institution: Vellore Institute of Technology, Chennai
- Subject Code: CSE4074
- Semester: Fall 2026-27
- Professor: Dr. S. Johanan Joysingh

## Publication note

A paper intended for publication is not included in this repository at this stage, in accordance with the review instructions.
