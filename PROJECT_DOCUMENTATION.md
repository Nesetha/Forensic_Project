# Project Documentation
## Explainable Procurement Fraud Investigation using Knowledge Graphs and Multi-Layer Risk Analytics

**Course:** Forensic Analytics  
**Institution:** Vellore Institute of Technology, Chennai  
**Subject Code:** CSE4074  
**Semester:** Fall 2026-27  
**Professor:** Dr. S. Johanan Joysingh  
**Team:** Nesetha J C, Hariharan M

---

## 1. Background and problem statement

Procurement records connect several entities, including purchase orders, suppliers, employees, invoices, and payments. Examining a transaction in isolation can reveal an unusual amount or date, but it may not show related transactions or recurring employee–supplier relationships. This project combines multiple forensic screening methods with relationship modelling and explainable scoring to prioritize cases for further review.

The goal is **investigation support**, not automatic fraud determination. A flag indicates an unusual pattern that needs contextual verification.

## 2. Objectives

- Prepare a real procurement dataset for analysis.
- Establish consistent identifiers and a purchase-order-level analytical model.
- Create controlled synthetic employee, invoice, and payment relationships to support methods that require those entities.
- Apply complementary forensic indicators.
- Represent transaction relationships as a knowledge graph.
- Combine indicators in an explainable investigation-priority score.
- Reconstruct selected cases and test intended signals against controlled scenarios.

## 3. Dataset and data preparation

The real procurement backbone is the **California Statewide Procurement and Contracting System (SCPRS) Purchase Order Data, 2012–2015**.

| Data preparation measure | Count |
|---|---:|
| Original records | 344,504 |
| Analytical line items after date filtering | 318,962 |
| Records removed by purchase-date validity filter | 25,542 |
| Unique purchase orders | 187,080 |
| Departments | 108 |
| Suppliers in the analytical subset | 24,260 |

The notebook profiles missingness and data types, converts dates and numeric fields, standardizes text and supplier codes, validates purchase dates within the study period, creates entity identifiers, and constructs a canonical PO-level baseline. Missing or invalid purchase dates are filtered because date-based analysis depends on valid chronology. Other missing fields are assessed according to their analytical role; missing monetary values are not simply replaced with zero.

## 4. Synthetic relationship data and controlled scenarios

The original source does not provide complete employee–invoice–payment relationships for this project. The notebook therefore adds controlled synthetic layers:

- **Employees:** 350 synthetic employees assigned across 108 departments.
- **Invoices:** one initial invoice per PO, followed by five intentionally duplicated invoices for the duplicate-invoice scenario.
- **Payments:** one payment per original invoice; the intentionally duplicated invoices have no corresponding payment.
- **Ground truth:** six scenario types with five controlled cases each, for 30 ground-truth rows.

The six scenario types are price manipulation, duplicate invoice, employee–vendor concentration, split purchase, temporal spike, and round-number/structuring. These injected cases are for implementation testing. They are not claims that the original SCPRS records contain confirmed fraud.

## 5. Forensic analytics

Eight complementary methods are used:

1. **Benford's Law:** compares first-digit frequencies with a reference distribution at the population level.
2. **Duplicate-invoice detection:** identifies repeated invoice patterns based on supplier identity, date, amount, and invoice identifiers.
3. **Statistical outlier analysis:** uses the interquartile range to flag unusually high PO values.
4. **Employee–vendor concentration analysis:** examines whether an employee interacts with a supplier disproportionately compared with an expected interaction level.
5. **Temporal anomaly analysis:** identifies unusually concentrated payment dates using a robust median/MAD baseline.
6. **PO–invoice price variance:** compares invoice amounts with associated PO values against a configured variance threshold.
7. **Split-purchase analysis:** searches for multiple sub-threshold purchases within a 30-day window that collectively exceed the configured threshold.
8. **Round-number analysis:** flags exact round monetary amounts as a supporting signal.

These methods are complementary rather than a sequential chain of proof. A flag can be legitimate; each result should be checked against source documents, approvals, contract amendments, invoice records, and payment evidence.

## 6. Knowledge graph and network context

The project uses NetworkX to create a directed graph with five node types: Employee, Supplier, Purchase Order, Invoice, and Payment. The relationships model who created a PO, which supplier supplied it, which invoice belongs to the PO, which payment settles an invoice, and which employee–supplier pair is associated with the transaction.

The recorded graph contains **585,855 nodes and 836,213 edges**. Forensic attributes are attached to relevant nodes so investigators can trace the evidence behind a flagged case. The graph supports relationship traceability and context; it is not a graph-neural-network fraud predictor.

## 7. Evidence convergence and risk scoring

The system counts concurrent risk indicators and calculates an explainable weighted score. The predefined component weights are:

| Component | Weight |
|---|---:|
| Price variance | 25 |
| Duplicate invoice | 20 |
| Employee–vendor concentration | 20 |
| Statistical outlier | 10 |
| Temporal anomaly | 10 |
| Split purchase | 10 |
| Round number | 5 |
| **Total theoretical maximum** | **100** |

The score is the sum of applicable weights, bounded from 0 to 100. It is a transparent heuristic score, not a trained model and not a fraud probability.

| Category | Score range |
|---|---:|
| Low | 0–19 |
| Moderate | 20–39 |
| High | 40–59 |
| Critical | 60–100 |

The observed maximum score is 60. Theoretical maximum and observed maximum are different and should not be conflated.

## 8. Overall results

The final PO-level risk assessment covers 187,080 purchase orders.

| Risk category | Purchase orders | Percentage |
|---|---:|---:|
| Low | 127,408 | 68.103% |
| Moderate | 58,246 | 31.134% |
| High | 1,425 | 0.762% |
| Critical | 1 | 0.001% |

These are investigation-priority categories, not verified fraud categories. A PO with no flags is not thereby proven legitimate, and a flagged PO is not thereby proven fraudulent.

## 9. Explainable case studies

### 9.1 Critical case: PO_051985

- **PO value:** $777
- **PO date:** 15 May 2014
- **Investigation score:** 60 — Critical
- **Primary risk drivers:** duplicate invoice, employee–vendor concentration, temporal anomaly, and split-purchase indicators.

The associated invoice and payment records show an invoice and payment amount of $777, with a 29-day payment lag. The employee–supplier relationship and nearby sub-threshold purchase activity contribute context to the score. The combination makes this case a high priority for detailed document review; it does not establish fraud.

### 9.2 High-priority case: PO_008062

- **PO value:** $1,000,000
- **PO date:** 31 October 2013
- **Investigation score:** 55 — High
- **Primary risk drivers:** statistical outlier, duplicate invoice, round-number amount, and employee–vendor concentration.

The associated invoice amount matches the PO amount, and the payment amount also matches. This does not remove the other screening signals, but it is important counter-context: the score reflects the combined indicators rather than a claim that the invoice and payment amounts disagree. The case warrants checking the underlying contract, approvals, supplier history, duplicate evidence, and payment documentation.

## 10. Controlled-case evaluation

The notebook evaluates 30 deliberately injected controlled cases across six scenario types. All 30 were detected through their intended primary forensic indicators, yielding a **100% controlled-case detection rate in this designed test**.

This result supports the implementation of the intended rules under controlled conditions. It must not be described as real-world fraud-detection accuracy, and it does not establish precision, recall, or false-positive rate across genuine procurement cases.

## 11. How to reproduce the workflow

1. Open `Forensic_Project.ipynb` in Google Colab.
2. Obtain the authorized SCPRS source CSV for the 2012–2015 period.
3. Run cells from top to bottom and upload the CSV when prompted.
4. Inspect cleaning, entity-resolution, synthetic-layer, forensic-analysis, graph, scoring, and evaluation outputs.
5. The final notebook cell writes the derived tables to CSV and packages them in a ZIP for download.

The notebook uses Colab-specific file upload and download utilities. Those cells need adaptation if running outside Google Colab. Install the packages listed in `requirements.txt` if they are not already available in the runtime.

## 12. Limitations and ethical interpretation

- Synthetic employee, invoice, payment, and scenario data are not verified source records.
- The six controlled scenarios test known patterns, not real-world generalization.
- Risk weights are predefined heuristics and have not been calibrated on labelled historical fraud cases.
- Benford deviation, large PO values, repeated amounts, temporal spikes, and employee–supplier concentration can occur in legitimate procurement.
- Before any operational conclusion, an investigator should inspect original documents, contracts, approvals, supplier records, and payment evidence.
- The project provides explainable screening and prioritization, not a legal determination of fraud.

## 13. Conclusion

The project demonstrates how multiple forensic screening methods can be combined with a knowledge graph, evidence convergence, and a transparent weighted score to prioritize procurement cases for review. The critical and high-priority examples show how the framework explains its risk drivers and links them to related entities. Controlled-case detection demonstrates that the intended rules identify the injected test patterns; real-world effectiveness would require expert review, labelled cases, false-positive analysis, and further validation.
