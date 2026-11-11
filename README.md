Cross-Artefact Forensic Timeline Triage
Correlating Windows Evidence into Prioritised Attack Chains for Faster Incident Investigation
Project status: Proposed / early development. The capabilities described below are design goals and are not yet implemented or validated.

Overview
Cross-Artefact Forensic Timeline Triage is a proposed open-source digital forensics framework for helping analysts identify and investigate significant activity within large Windows forensic timelines.

Existing tools such as Plaso and Hayabusa can produce extensive event timelines and detection results. These outputs are valuable, but analysts may still need to manually compare records from different sources, determine which events are related, and decide what to investigate first.

This project aims to bridge that gap by ingesting existing timeline and detection outputs, converting them into a common event schema, correlating relevant events into potential attack chains, and generating a prioritised, explainable case summary. It is intended to assist human investigators, not replace forensic judgment or serve as a standalone proof of compromise.
Problem Statement
Windows investigations can involve hundreds of thousands of records across multiple artefacts. Relevant evidence may be scattered between event logs, registry artefacts, filesystem metadata, application execution traces, and browser activity. The main challenges are:
- Volume: Large timelines make manual triage time-consuming.
- Fragmentation: Different tools and artefact types represent activity using different fields, timestamps, and formats.
- Context: A suspicious event in isolation may be ambiguous; related events across sources can provide useful context.
- Prioritisation: Analysts need a defensible way to focus on likely-relevant sequences without discarding other evidence.
- Explainability: Findings should show which underlying records and correlation rules support a conclusion.
Project Objectives
1. Normalise supported timeline and detection outputs into a common event schema.
2. Correlate events across Windows artefacts using transparent rules and temporal relationships.
3. Reconstruct candidate attack chains and map supported behaviours to MITRE ATT&CK.
4. Prioritise findings with an interpretable scoring model based on rule severity, cross-source corroboration, and temporal and rarity-related signals where measurable.
5. Summarise and trace evidence so analysts can inspect each chain's supporting events and original source records.
6. Evaluate effectiveness against controlled attack simulations, known ground truth, and suitable public forensic datasets.
Proposed Scope
Inputs and evidence sources
The framework is intended to consume outputs from existing forensic collection, parsing, and detection tools, rather than directly acquire evidence from live systems. Initial input support will be defined during development and may include:
Source or format	Example evidence of interest	Planned handling
Plaso timeline exports	File activity, application and OS artefacts represented in a timeline	Parse supported export formats and map fields into the common schema
Hayabusa results	Windows Event Log detections and related metadata	Import supported result formats and preserve detection context
Windows Event Logs	Process creation, logon activity, service changes, log clearing	Consume events through parsed timeline or detection outputs
Registry-derived records	Autoruns, persistence-related configuration changes	Normalise supported registry artefact events
Filesystem records	File creation/modification and possible staging activity	Normalise supported filesystem timeline events
Prefetch records	Program execution indicators	Integrate parsed execution-related events
Browser records	Browsing and download activity	Integrate parsed browser events where available


Note: This table defines intended coverage, not a claim that each source is already supported. Exact adapters and fields will depend on the available export formats and test data.
Out of scope for the initial prototype
- Live endpoint monitoring and automated evidence acquisition.
- Automatic attribution of attackers or definitive incident conclusions.
- Autonomous response or changes to investigated systems.
- Guaranteed detection of all attack techniques.
Proposed Workflow
Existing forensic tools / parsed artefact outputs
           |
           v
   Input adapters and validation
           |
           v
  Unified event schema + time normalisation
           |
           v
 Rule-based cross-artefact correlation
           |
           v
  Candidate attack-chain reconstruction
           |
           v
  Explainable scoring and prioritisation
           |
           v
 Case summary + supporting-event references
           |
           v
          Analyst review
1. Ingestion and normalisation
Parse selected output formats, validate mandatory fields, and map tool-specific fields into a shared representation. The proposed schema will record fields such as:
- Event identifier and original record reference.
- Source tool, artefact type, and source file.
- Timestamp, original timezone or offset where known, and normalised UTC time.
- Host, user, process, file path, and other relevant entities when available.
- Event action, detection metadata, and source confidence or parsing notes.
The system should retain enough original metadata to let an investigator trace a finding back to its source. Timestamp ambiguity, missing fields, and differing timestamp semantics must be identified rather than silently treated as exact matches.
2. Cross-artefact correlation
Apply documented, configurable rules that link events through shared entities and context, such as a host or user identity, file path, process reference, and a defined time window. Rules will be designed to reduce unsupported links and expose the reasoning behind each correlation.
3. Candidate attack chains
Group related events into candidate sequences that may correspond to adversary activity. Early example scenarios for rule development include:
- Suspicious program execution.
- Persistence-related changes.
- Possible data staging activity.
- Anti-forensic activity, such as relevant event-log clearing.
Where appropriate, behaviours will be mapped to MITRE ATT&CK techniques. An ATT&CK mapping describes a possible behavioural relationship; it does not establish that an attack occurred.
4. Explainable prioritisation
Explore a configurable scoring approach incorporating:
- Rule severity: How significant the matched behaviour is under the configured rules.
- Cross-source corroboration: Whether independent artefact sources support related activity.
- Temporal relationships: Whether event ordering and proximity match the rule's expected sequence.
- Rarity or deviation: Whether a feature is unusual within an explicitly defined comparison dataset, where sufficient data exists.
Scores are intended to support relative triage, not represent calibrated probabilities of malicious activity. Each prioritised chain should show its triggered rules, evidence links, contributing factors, and key uncertainties.
5. Investigator-facing results
The intended output is a compact case summary containing candidate attack chains, ordered by priority, with a short explanation, relevant timestamps, ATT&CK mappings when supported, and references to supporting source records. Analysts should be able to inspect the evidence behind a finding instead of relying only on a numeric score.
Evaluation Plan
The prototype will be assessed in an isolated, virtualised lab using controlled attack scenarios with documented ground truth, supplemented by suitable publicly available forensic datasets.
Measure	What it evaluates
Precision	Proportion of flagged events or chains that match the defined ground truth
Recall	Proportion of ground-truth relevant activity that is successfully identified
Attack-chain reconstruction quality	Correct event membership and sequence relationships compared with annotated ground truth
Analyst review workload	Number or proportion of events requiring review compared with an unprioritised timeline
Baseline comparison	Performance against selected existing single-source detections or timeline review workflows
Processing performance	Runtime and resource usage on datasets of different sizes


Evaluation definitions, datasets, thresholds, and baselines will be documented before results are reported. No performance claims or reduction in investigation time are made at this stage. Any claim about time savings will require a suitable study or a carefully qualified workload proxy.
Development Roadmap
- [ ] Define the minimum viable scope and supported input formats.
- [ ] Specify the canonical event schema and timestamp handling rules.
- [ ] Implement an initial Plaso export adapter.
- [ ] Implement an initial Hayabusa result adapter.
- [ ] Add schema validation and source-record traceability.
- [ ] Implement and unit-test the first correlation rules.
- [ ] Add candidate chain grouping and ATT&CK mappings.
- [ ] Implement explainable priority scoring.
- [ ] Generate a case summary and evidence-backed findings.
- [ ] Build controlled lab scenarios and curate test datasets.
- [ ] Evaluate detection, reconstruction, review workload, and performance.
- [ ] Publish reproducible setup instructions, example data, and results.
Getting Started
The project is currently in the design and planning phase. There is no runnable release or installation procedure yet. This section will be updated with prerequisites, installation commands, sample inputs, and usage examples once the initial prototype is available.
Design Principles
- Forensic traceability: Preserve references to original inputs and avoid silent alteration of evidence.
- Explainability: Make rules, links, scores, and uncertainties inspectable.
- Reproducibility: Version rules and document test conditions.
- Modularity: Separate ingestion, normalisation, correlation, scoring, and reporting.
- Responsible handling: Use sanitised or appropriately authorised forensic datasets and avoid committing sensitive case data.
Contributing
This is an early-stage project. Feedback on the proposed scope, event schema, correlation methods, and evaluation approach is welcome. Contribution guidance and development setup instructions will be added as the codebase develops.
License
The project is intended to be released under the MIT License. See LICENSE once it has been added to the repository. Dependencies and incorporated datasets may have separate terms.
Disclaimer
This project is intended for digital forensic research, education, and authorised incident investigation. Its outputs are decision-support indicators and must be independently assessed by a qualified investigator. The framework is under development and should not currently be relied upon for operational forensic conclusions.
