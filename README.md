# AI Proposal Triage Design

**Candidate:** Dharam Dan  
**Assessment reference:** MATACS-TH-2026-02  
**Time actually spent:** 220 Minutes  
**Submission:** Associate Software Developer technical design exercise

## Overview

A design for evaluating contractor proposals using AI only to extract cited facts, configurable rules to check eligibility and score proposals, and a human reviewer to decide the final award. The proposed workflow protects against missing evidence and instructions embedded in untrusted proposal text.

## Required deliverables

- [Process-flow diagram](diagram/process-flow.png)
- [Five design answers, evaluation policy and assumptions](WRITEUP.md)
- [Exact AI extraction prompts](PROMPTS.md)
- [Sample expected ranking](ranking.json)

## AI tool usage

## AI Tool Usage

I used ChatGPT to help structure the workflow, develop the scoring approach, and identify potential risks. I corrected ChatGPT's initial process-flow diagram because it incorrectly routed proposals that failed mandatory criteria to the human approval stage. I revised the flow to mark confirmed failures as ineligible and record them in the audit log, while flagging missing information for human review. I also independently calculated the proposal scores in WRITEUP.md and checked the deliverables against the assessment brief.
