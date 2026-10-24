# Explainable Procurement Fraud Investigation using Knowledge Graphs and Multi-Layer Risk Analytics

Developed as part of the course on "Forensic Analytics" at Vellore Institute of Technology, Chennai.

by **Nesetha J C, Hariharan M**

**Subject Code:** CSE4074  
**Semester:** Fall 2026-27  
**Professor:** Dr. S. Johanan Joysingh

---

## 1. Project Overview

Procurement systems generate interconnected records involving purchase orders (POs), suppliers, employees, invoices, and payments. Analysing transactions independently may fail to reveal unusual financial patterns, repeated billing, concentrated employee–supplier relationships, or suspicious timing patterns.

This project presents an explainable procurement fraud investigation framework that combines forensic analytics, knowledge graph modelling, evidence convergence, and multi-layer risk scoring. The framework examines procurement records using multiple complementary indicators and identifies cases that warrant further investigation. It also provides traceability between related entities and explains why a transaction receives a particular investigation-priority score.

The system is designed as an investigation-support tool. A flagged transaction is not automatically fraudulent, and the risk score is not a probability of fraud.

## 2. Problem Statement

Traditional transaction-level analysis may identify unusual purchase values or invoice amounts but can overlook relationships among employees, suppliers, purchase orders, invoices, and payments. Investigators need a systematic way to combine financial, behavioural, and temporal indicators with relationship context and explain why particular cases should be prioritised for review.

This project addresses that need through complementary forensic screening methods, a directed knowledge graph, evidence convergence, and transparent weighted risk scoring.

## 3. Objectives

- Analyse and prepare real procurement data for forensic investigation.
- Create consistent entity identifiers and a purchase-order-level analytical model.
- Add clearly identified synthetic employee, invoice, and payment layers where the source data lacks the required relationships.
- Implement eight complementary forensic screening methods.
- Create a knowledge graph connecting employees, suppliers, purchase orders, invoices, and payments.
- Combine multiple indicators through evidence convergence and weighted investigation-priority scoring.
- Explain selected critical and high-priority cases using their contributing indicators and related entities.
- Evaluate the intended detection rules using controlled scenarios with known ground truth.

## 4. Dataset and Data Preparation

### 4.1 Original Dataset

The project uses the **California Statewide Procurement and Contracting System (SCPRS) Purchase Order Data, 2012–2015** as its real procurement data backbone.

The original dataset contains 344,504 records and 32 columns. Relevant information includes purchase-order details, supplier information, department, quantity, unit price, total price, purchase date, and fiscal year.

| Dataset measure | Count |
|---|---:|
| Original records | 344,504 |
| Original columns | 32 |
| Analytical line items after purchase-date filtering | 318,962 |
| Records removed by purchase-date filtering | 25,542 |
| Unique purchase orders | 187,080 |
| Departments | 108 |
| Suppliers in the analytical subset | 24,260 |

### 4.2 Data Preparation

The notebook profiles the data, processes date and numeric fields, validates identifiers, handles missing values according to their analytical role, and creates consistent identifiers for line items, purchase orders, departments, and suppliers.

Records with missing or invalid purchase dates were excluded because temporal analysis, chronological relationships, and 30-day split-purchase checks require valid dates. Missing monetary values are not automatically replaced with zero.

### 4.3 Synthetic Data Layers

The original procurement data does not contain all the employee, invoice, and payment relationships needed for the proposed investigation workflow. Therefore, controlled synthetic layers were created:

- 350 synthetic employees across 108 departments.
- Invoice records linked to purchase orders.
- Payment records linked to original invoices.
- Five additional duplicate invoices for controlled testing.
- A separate ground-truth registry for six controlled scenario types, with five cases per scenario.

These synthetic layers support relationship-based investigation and controlled testing. They are not original SCPRS records and are not evidence of confirmed real-world fraud.

The original dataset is not included in this repository. Users should obtain it from an authorised source and follow the applicable access and redistribution terms.

## 5. Methodology

The project follows these major stages:

1. **Data preparation:** profile, clean, and validate the procurement dataset.
2. **Entity resolution:** create stable identifiers and construct the purchase-order-level baseline.
3. **Synthetic data generation:** construct employee, invoice, and payment relationships.
4. **Controlled scenario generation:** introduce six types of test scenarios with separate ground-truth records.
5. **Forensic analytics:** run eight complementary screening methods.
6. **Knowledge graph construction:** model entities and their relationships using NetworkX.
7. **Evidence convergence and risk scoring:** combine applicable indicators using predefined weights.
8. **Case investigation and evaluation:** explain selected cases and assess the rules against controlled ground truth.

## 6. Forensic Analytics

The framework implements eight complementary methods.

| Method | Purpose |
|---|---|
| Benford's Law | Examines population-level deviations in leading-digit distributions. |
| Duplicate-invoice detection | Identifies repeated supplier, invoice-date, and invoice-amount combinations. |
| IQR statistical outlier analysis | Identifies unusually high purchase-order values relative to the population. |
| Employee–vendor concentration analysis | Detects disproportionate employee–supplier interaction patterns. |
| Temporal anomaly analysis | Identifies unusual concentrations of payments on particular dates. |
| PO–invoice price variance | Flags invoice amounts that differ substantially from associated PO values. |
| Split-purchase analysis | Identifies related sub-threshold purchases that collectively exceed a configured threshold within a time window. |
| Round-number analysis | Flags exact round monetary amounts as a supporting screening signal. |

These methods identify patterns that may require further investigation. Legitimate operational explanations must be considered before drawing conclusions.

## 7. Knowledge Graph

A directed graph was implemented using NetworkX to represent five entity types:

- Employee
- Supplier
- Purchase Order
- Invoice
- Payment

The graph contains the following principal relationships:

- Employee → Purchase Order (`CREATED`)
- Purchase Order → Supplier (`SUPPLIED_BY`)
- Purchase Order → Invoice (`HAS_INVOICE`)
- Invoice → Payment (`PAID_BY`)
- Employee → Supplier (`INTERACTS_WITH`)

The recorded graph contains **585,855 nodes and 836,213 edges**. Forensic indicators are attached to relevant nodes to support relationship tracing and explainable case investigation.

The graph provides relationship context; it is not a graph-neural-network fraud prediction model.

## 8. Evidence Convergence and Risk Scoring

The framework combines applicable indicators through predefined heuristic weights.

| Risk component | Weight |
|---|---:|
| PO–invoice price variance | 25 |
| Duplicate invoice | 20 |
| Employee–vendor concentration | 20 |
| Statistical outlier | 10 |
| Temporal anomaly | 10 |
| Split purchase | 10 |
| Round-number signal | 5 |
| **Theoretical maximum** | **100** |

The final score is the sum of applicable component weights, bounded between 0 and 100. The weights are predefined and are not learned from labelled fraud data. They do not represent probabilities.

| Investigation-priority category | Score range |
|---|---:|
| Low | 0–19 |
| Moderate | 20–39 |
| High | 40–59 |
| Critical | 60–100 |

The observed maximum score is 60. It is not rescaled to the theoretical maximum of 100.

## 9. Results and Discussion

The final risk assessment covers **187,080 unique purchase orders**.

| Category | Purchase orders | Percentage |
|---|---:|---:|
| Low | 127,408 | 68.103% |
| Moderate | 58,246 | 31.134% |
| High | 1,425 | 0.762% |
| Critical | 1 | 0.001% |

### Selected Investigation Cases

**Critical case — PO_051985**

- PO value: $777
- Investigation-priority score: 60
- Category: Critical
- Contributing indicators: duplicate invoice, employee–vendor concentration, temporal anomaly, and split purchase.

This case is prioritised because several indicators converge. The score does not establish fraud, and the associated transactions require documentary verification.

**High-priority case — PO_008062**

- PO value: $1,000,000
- Investigation-priority score: 55
- Category: High
- Contributing indicators: statistical outlier, duplicate invoice, round-number amount, and employee–vendor concentration.

This case warrants review of the procurement documents, duplicate evidence, supplier relationship, approvals, and payment records.

### Controlled-Case Evaluation

All **30 deliberately injected controlled cases** were detected through their intended primary forensic indicators, giving a 30/30 or 100% controlled-case detection rate in this designed experiment.

This result validates the intended rules under controlled conditions. It is not real-world fraud-detection accuracy and does not establish precision, recall, or the false-positive rate on genuine procurement cases.

## 10. Technologies Used

- Python
- Google Colab
- Pandas and NumPy
- NetworkX
- Matplotlib and Seaborn, where used for visualisation
- CSV-based data processing
- Markdown documentation

## 11. Repository Contents

- `Forensic_Project.ipynb` — the Google Colab notebook containing data preparation, synthetic data generation, forensic analysis, graph construction, risk scoring, case investigation, evaluation, and data export.
- `README.md` — project overview, methods, results, setup instructions, and limitations.
- `PROJECT_DOCUMENTATION.md` — detailed explanation of the project workflow and findings.
- `requirements.txt` — principal Python dependencies.

## 12. How to Run

1. Open `Forensic_Project.ipynb` in Google Colab.
2. Obtain the authorised SCPRS Purchase Order dataset for 2012–2015.
3. Run the notebook cells in order and upload the source CSV when prompted.
4. Review the data preparation, entity-resolution, synthetic-data, forensic-analysis, graph, scoring, and evaluation outputs.
5. Run the final export cell to generate a ZIP file containing derived analytical CSV files.

The notebook uses Google Colab upload and download utilities. Running it in a local Jupyter environment may require adapting those cells. Install the required dependencies from `requirements.txt` if they are not already available.

## 13. Limitations and Responsible Interpretation

- Employee, invoice, payment, and injected scenario data are synthetic additions, not verified original SCPRS relationships.
- Controlled scenarios test the intended rules but do not demonstrate performance on genuine fraud investigations.
- Risk weights are heuristic and require expert review and sensitivity analysis.
- High values, duplicate patterns, round amounts, temporal spikes, Benford deviations, and concentrated employee–supplier relationships can have legitimate explanations.
- The framework does not establish intent, collusion, financial loss, or legal wrongdoing.
- A flagged transaction is not automatically fraudulent, and a transaction with no flags is not guaranteed to be legitimate.

## 14. Team and Course Details

- **Team members:** Nesetha J C, Hariharan M
- **Course:** Forensic Analytics
- **Institution:** Vellore Institute of Technology, Chennai
- **Subject Code:** CSE4074
- **Semester:** Fall 2026-27
- **Professor:** Dr. S. Johanan Joysingh

## 🌐 Live Project Portfolio

**[Visit the Forensic Analytics Portfolio](https://nesetha.github.io/Forensic_Project/)**

Explore the project overview, forensic methodology, knowledge graph, risk analysis, selected investigation cases, and evaluation results.
