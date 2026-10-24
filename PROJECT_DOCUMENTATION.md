# Project Documentation

# Explainable Procurement Fraud Investigation using Knowledge Graphs and Multi-Layer Risk Analytics

**Developed as part of the course on "Forensic Analytics" at Vellore Institute of Technology, Chennai.**

**Team:** Nesetha J C, Hariharan M  
**Subject Code:** CSE4074  
**Semester:** Fall 2026-27  
**Professor:** Dr. S. Johanan Joysingh

---

## 1. Project Background

Procurement processes generate connected records for purchase orders (POs), suppliers, employees, invoices, and payments. A transaction may appear ordinary when examined individually, while its relationships with other purchases, suppliers, invoices, or payment dates may reveal additional patterns.

Analysing each indicator independently can miss relationships across the procurement process. This project develops an explainable forensic-investigation workflow that combines multiple screening methods, a knowledge graph, evidence convergence, and transparent risk scoring.

The purpose is to help an analyst determine which records should be reviewed first and understand the indicators contributing to that prioritisation.

**The system does not automatically declare fraud.** An anomaly is a lead for investigation, not proof of wrongdoing. Confirmation requires supporting documents, approval records, contracts, invoices, payment evidence, and human review.

## 2. Problem Statement

Transaction-level procurement checks can identify unusual amounts, repeated invoice details, or unexpected timing. However, individual indicators may not reveal the relationships among employees, suppliers, purchase orders, invoices, and payments.

There is a need for an explainable approach that combines financial, behavioural, temporal, and relationship-based evidence, prioritises cases for investigation, and preserves the distinction between a suspicious pattern and confirmed fraud.

This project addresses the problem through complementary forensic rules, a directed knowledge graph, evidence convergence, and a weighted investigation-priority score.

## 3. Objectives

1. Prepare and validate a real procurement dataset for forensic screening.
2. Create stable entity identifiers and a canonical purchase-order-level analytical table.
3. Add clearly identified synthetic employee, invoice, and payment relationships where the original source data is incomplete for the workflow.
4. Implement complementary forensic screening methods.
5. Model relevant procurement entities and relationships in a directed knowledge graph.
6. Measure evidence convergence and calculate a transparent weighted investigation-priority score.
7. Explain selected high-priority cases by tracing their related entities and contributing indicators.
8. Test the intended primary rules against controlled scenarios with known ground truth.

## 4. Dataset and Preparation

### 4.1 Original Procurement Dataset

The real procurement backbone is the **California Statewide Procurement and Contracting System (SCPRS) Purchase Order Data, 2012–2015**. It contains procurement records and fields such as purchase-order information, supplier, department, quantity, unit price, total price, and purchase date.

| Preparation measure | Count |
|---|---:|
| Original records | 344,504 |
| Original columns | 32 |
| Analytical line items after date filtering | 318,962 |
| Records removed by purchase-date validity filtering | 25,542 |
| Unique purchase orders in the analytical subset | 187,080 |
| Departments | 108 |
| Suppliers in the analytical subset | 24,260 |

A source line item is not necessarily a unique purchase order because one PO may contain multiple line items. Therefore, the analytical line-item count differs from the number of unique POs.

### 4.2 Data Cleaning and Validation

The notebook profiles columns, data types, missingness, and procurement values. It converts date and numeric fields, normalises relevant text and supplier identifiers, checks entity mappings, and creates stable identifiers for line items, POs, departments, and suppliers.

Records with missing or invalid purchase dates are excluded from the analytical subset because temporal analysis, 30-day split-purchase windows, and chronological invoice/payment relationships require valid dates. Imputing arbitrary dates could introduce artificial timing evidence.

Other missing fields are assessed according to their analytical role. Missing monetary values are not automatically replaced with zero.

The resulting PO-level baseline supports the downstream synthetic layers and forensic analysis. Data-quality patterns, such as multiple source lines or multiple associated values, are treated as context rather than fraud findings by themselves.

## 5. Synthetic Layers and Controlled Test Scenarios

### 5.1 Why Synthetic Data Was Needed

The original SCPRS procurement data does not contain the complete employee–invoice–payment relationship structure needed for every part of the proposed workflow.

The project therefore constructs controlled synthetic relationship layers around the real procurement backbone. This allows relationship-based analytics and end-to-end case tracing to be tested while retaining the original dataset's procurement structure.

Synthetic records must be distinguished from original government records throughout the project.

### 5.2 Synthetic Entities

- **Employees:** 350 synthetic employee records are assigned across 108 departments. All 187,080 POs are assigned to an employee in the constructed layer.
- **Invoices:** one initial invoice is created per PO. Five additional duplicate invoices are introduced for controlled duplicate-invoice testing, resulting in 187,085 invoice records.
- **Payments:** one payment is created for each original invoice, resulting in 187,080 payments. The five intentionally duplicated invoices have no corresponding payments.
- **Ground-truth registry:** a separate registry records six controlled scenario types, with five cases per scenario, for 30 ground-truth rows. Some scenarios overlap on entity IDs, so the 30 rows represent 27 unique entity IDs.

### 5.3 Controlled Scenarios

Six controlled scenario types were introduced:

1. **Price manipulation:** selected invoice amounts are modified relative to their associated PO values.
2. **Duplicate invoice:** additional invoice records reproduce existing invoice details.
3. **Employee–vendor concentration:** selected employee–supplier interactions are deliberately made unusually concentrated.
4. **Split purchase:** related purchases below a configured threshold are grouped within a 30-day window so that their combined value exceeds the threshold.
5. **Temporal spike:** selected payments are placed on a date with unusually high payment activity.
6. **Round number / structuring:** selected amounts are adjusted to exact round-number values.

These scenarios are deliberately constructed tests. They are not discoveries of confirmed fraud in the original SCPRS data. The ground-truth labels are kept separate from risk-scoring features so that the labels are not used as evidence in the risk score.

## 6. Forensic Analytics

The project implements eight complementary methods. Their outputs are screening signals that are combined later; none individually proves fraud.

### 6.1 Benford's Law

Benford's Law provides a reference distribution for the leading digits of many naturally occurring numerical datasets. The project compares the first-digit distribution of positive PO values with the expected distribution and uses a chi-square goodness-of-fit statistic to assess population-level deviation.

The observed distribution differs from the Benford reference. This indicates a statistical deviation at the population level; it does not identify an individual fraudulent PO or establish fraud.

### 6.2 Duplicate-Invoice Detection

The refined rule examines positive invoice records with an identified supplier and looks for repeated supplier, invoice-date, and invoice-amount combinations associated with multiple invoice IDs.

The method identifies candidate duplicate groups. Repeated invoice details may result from legitimate corrections, resubmissions, or data-processing procedures. Original invoice documents, supplier records, and payment evidence must therefore be checked.

### 6.3 IQR Statistical Outlier Analysis

The interquartile range (IQR) method identifies unusually high PO values relative to the distribution of positive PO amounts.

This method is useful for screening extreme values. However, large government purchases can be legitimate, and an outlier is a statistical description rather than a fraud label.

### 6.4 Employee–Vendor Concentration Analysis

This method groups employee–supplier interactions and compares observed activity with an expected interaction level. Relationships meeting the configured minimum activity, interaction-ratio, and excess-interaction criteria are flagged for contextual review.

Concentration can reflect normal job responsibilities or purchasing patterns. It is not proof of collusion or fraud.

### 6.5 Temporal Anomaly Analysis

Daily payment counts are compared with a robust baseline using the median and median absolute deviation (MAD). Dates exceeding the configured robust-score threshold are flagged for review.

A payment spike may have legitimate causes, including payment schedules, month-end processing, or batch payments. The date and related transactions must be examined before drawing conclusions.

### 6.6 PO–Invoice Price Variance

Invoice amounts are compared with their associated PO values. The configured screening threshold flags a variance outside ±25%.

The rule highlights financial inconsistencies for review. Taxes, approved amendments, quantity changes, or other documented adjustments may explain a difference.

### 6.7 Split-Purchase Analysis

This method searches for multiple positive, sub-threshold POs associated with the same employee and identified supplier within a 30-day window, where the combined value exceeds the configured $10,000 threshold.

This may identify a purchasing pattern worth checking against procurement rules, approval limits, and authorisation records. A qualifying group is not automatically a violation.

### 6.8 Round-Number Analysis

The method flags exact round monetary amounts, with exact $1,000 multiples used as the primary signal.

Round values are common in ordinary purchasing. This method is therefore treated as supporting evidence and should not be used alone to classify a transaction as fraudulent.

## 7. Knowledge Graph and Network Analysis

### 7.1 Graph Model

The project uses a directed NetworkX graph to represent five node types and five principal relationship types.

| Relationship | Meaning |
|---|---|
| Employee → Purchase Order (`CREATED`) | Associates an employee with a PO |
| Purchase Order → Supplier (`SUPPLIED_BY`) | Links a PO to its supplier |
| Purchase Order → Invoice (`HAS_INVOICE`) | Links a PO to an invoice |
| Invoice → Payment (`PAID_BY`) | Links an invoice to its payment |
| Employee → Supplier (`INTERACTS_WITH`) | Represents an employee–supplier interaction |

The recorded graph contains **585,855 nodes and 836,213 edges**. Its node counts are:

| Node type | Count |
|---|---:|
| Invoice | 187,085 |
| Payment | 187,080 |
| Purchase Order | 187,080 |
| Supplier | 24,260 |
| Employee | 350 |
| **Total** | **585,855** |

Graph integrity checks validate the expected entity sets and key relationships. Supplier nodes without a transaction relationship are documented rather than automatically treated as errors.

### 7.2 Purpose of the Graph

The graph allows an analyst to trace a PO to its employee, supplier, invoice, and payment and to examine employee–supplier relationships in context. Forensic indicators are attached as attributes to relevant nodes so the reasons for prioritising a selected case can be inspected.

The project uses graph relationships for context and traceability. It does not use a graph neural network or claim that graph connectivity alone predicts fraud.

## 8. Evidence Convergence and Risk Scoring

### 8.1 Evidence Convergence

Each PO is assessed for the presence of applicable forensic indicators. Counting concurrent indicators helps distinguish records with multiple screening signals from those with fewer signals.

The full PO population has the following convergence distribution:

| Number of indicators | Purchase orders |
|---|---:|
| 0 | 89,220 |
| 1 | 71,027 |
| 2 | 23,769 |
| 3 | 2,887 |
| 4 | 177 |
| **Total** | **187,080** |

A PO with zero indicators is not proven legitimate. Multiple indicators provide stronger reasons for review but do not prove fraud.

### 8.2 Weighted Investigation-Priority Score

The framework uses predefined, transparent heuristic weights.

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

The final score is the sum of applicable component weights, bounded between 0 and 100. The weights were not learned from labelled fraud examples and are not probabilities. The observed maximum score is 60; it should not be rescaled to the theoretical maximum of 100.

| Priority category | Score range |
|---|---:|
| Low | 0–19 |
| Moderate | 20–39 |
| High | 40–59 |
| Critical | 60–100 |

The score represents **investigation priority**, not the probability that fraud occurred.

## 9. Results and Discussion

### 9.1 Overall Risk Distribution

The final risk assessment covers 187,080 unique POs.

| Category | Purchase orders | Percentage |
|---|---:|---:|
| Low | 127,408 | 68.103% |
| Moderate | 58,246 | 31.134% |
| High | 1,425 | 0.762% |
| Critical | 1 | 0.001% |

These categories represent investigation priority only. The relatively small High and Critical groups provide a focused starting point for manual review; they are not counts of confirmed fraud cases.

### 9.2 Critical Case: PO_051985

| Field | Case detail |
|---|---|
| PO value | $777 |
| PO date | 15 May 2014 |
| Employee | `EMP_00163` |
| Supplier | `VENDOR_1755386` |
| Score | 60 — Critical |
| Contributing indicators | Duplicate invoice, employee–vendor concentration, temporal anomaly, split purchase |

The associated invoice (`INV_051986`) is dated 13 June 2014 and has an amount of $777. The associated payment (`PAY_051986`) is dated 12 July 2014 and also has an amount of $777, with a 29-day payment lag after the invoice.

The employee–supplier relationship shows 42 POs, an interaction ratio of approximately 7.01, and excess interactions of approximately 36.01 under the model's expectation calculation. The split-purchase context includes 11 qualifying sub-$10,000 POs within the configured time context, totalling $19,227.78.

These details explain why the case receives a high investigation priority. They do not establish that the PO or the related purchases are fraudulent. Recommended follow-up includes checking original invoices, purchase approvals, supplier records, contract terms, payment documentation, and the reasons for the repeated or concentrated activity.

### 9.3 High-Priority Case: PO_008062

| Field | Case detail |
|---|---|
| PO value | $1,000,000 |
| PO date | 31 October 2013 |
| Employee | `EMP_00161` |
| Supplier | `VENDOR_1106502` |
| Score | 55 — High |
| Contributing indicators | Statistical outlier, duplicate invoice, round-number amount, employee–vendor concentration |

The associated invoice (`INV_008063`) is dated 22 November 2013 for $1,000,000. The recorded invoice amount matches the PO, and the payment amount also matches.

The duplicate, outlier, round-number, and relationship indicators still warrant review, but the amount match is important context and should not be described as a price discrepancy. The investigator should inspect duplicate evidence, the contract and approvals, supplier history, and payment documentation.

### 9.4 Controlled-Case Evaluation

The six controlled scenarios contain five cases each, for 30 ground-truth rows. All 30 were detected by their intended primary forensic indicators, yielding a **100% controlled-case detection rate in this designed experiment**.

This result validates that the corresponding rules identified the injected patterns under controlled conditions. It is not real-world fraud-detection accuracy and does not establish precision, recall, or the false-positive rate on genuine procurement cases.

## 10. Reproducibility and Repository Files

The repository contains the Google Colab notebook, the main README, this documentation file, and a dependency list.

To reproduce the workflow:

1. Open `Forensic_Project.ipynb` in Google Colab.
2. Obtain the authorised SCPRS source CSV for the 2012–2015 period.
3. Run the cells in order and upload the source file when prompted.
4. Review the data-cleaning, entity-resolution, synthetic-layer, forensic-analysis, graph, scoring, and evaluation outputs.
5. Run the final export cell to generate a ZIP containing derived analytical CSV files.

The notebook uses Google Colab-specific upload and download utilities. Running it locally requires adapting those calls. Install the dependencies listed in `requirements.txt` if they are not already available.

## 11. Limitations and Responsible Interpretation

- Employee, invoice, payment, and injected scenario data are synthetic additions, not verified original SCPRS relationships.
- The injected cases are controlled tests and do not demonstrate performance on genuine fraud investigations.
- Risk weights are heuristic and should be reviewed by domain experts and tested through sensitivity analysis.
- Benford deviations, large values, repeated invoice details, round amounts, temporal spikes, and concentrated employee–supplier relationships can have legitimate explanations.
- The framework does not establish intent, collusion, financial loss, or legal wrongdoing.
- A flagged record requires investigation; an unflagged record is not guaranteed to be legitimate.

## 12. Conclusion

This project demonstrates a structured way to combine procurement data preparation, multiple forensic screening methods, relationship modelling, evidence convergence, and transparent risk scoring.

The knowledge graph supports traceability between employees, suppliers, POs, invoices, and payments, while the selected case studies demonstrate how individual risk drivers can be explained. The controlled test detected all 30 injected cases through their intended primary indicators, but this result is limited to the designed experiment.

Operational use would require expert validation, genuine labelled cases, false-positive analysis, sensitivity testing, and review of supporting documents.
