# Itinera AI: Constraint-Based Itinerary Builder

## System Title and Theme
**Itinera AI** — an intelligent itinerary builder that recommends specific places to visit and schedules them into a feasible plan based on a user’s time window, budget, interests, and travel constraints.

## System Overview (≤ 250 words)
Itinera AI generates short, realistic itineraries such as “Charlotte from 1pm–5pm under $30 with coffee + parks + local food” or “Phnom Penh day plan under $60.” Users provide a destination, a start/end time, a budget, interests (e.g., coffee, museums, nature, shopping, local food), and constraints (walking/short rides, must-include POIs, avoid categories). The system selects concrete points of interest (POIs), orders them into a schedule that fits the time window, estimates cost and travel/visit time, and explains why each stop was chosen.

This theme fits AI because itinerary building requires behaviors that appear intelligent: representing preferences and constraints, retrieving relevant candidates from a knowledge base, using heuristic scoring to trade off competing goals, and searching for a feasible plan under time/budget constraints. The design is intentionally modular and testable: each module has explicit inputs and outputs, deterministic core logic where possible, and evaluation metrics (budget use, time feasibility, constraint satisfaction). The initial system will use a curated POI dataset in JSON/CSV for a small set of cities (e.g., Charlotte and one Cambodia city). If time allows, a later extension can import POIs from a public source, but the core system does not depend on external APIs.

---

## Module Descriptions (6 modules)

### Module 1: TripSpec Parser & Preference Model
**Topics:** Knowledge representation  
**Input:** Raw user request (city, start/end time, budget, interests, constraints, starting location optional)  
**Output:** `TripSpec` object (normalized time window, numeric budget, weighted interest vector, hard constraints)  
**Integration:** Produces the single structured spec consumed by all modules  
**Prerequisites:** None beyond basic Python  
This module validates and normalizes user input (e.g., parses “1pm–5pm,” converts budget to float, assigns default weights for interests). It encodes hard constraints (must include / must avoid, transport mode) and produces a canonical representation for consistent downstream logic.

### Module 2: POI Knowledge Base (Dataset + Index)
**Topics:** Knowledge bases, representation  
**Input:** POI dataset (JSON/CSV) containing name, tags, estimated cost, coordinates (or neighborhood), typical visit duration, and hours/open rules  
**Output:** In-memory POI store with query/filter functions  
**Integration:** Supplies candidate places and metadata used by scoring, planning, and estimation  
**Prerequisites:** Module 1 (city chosen)  
This module loads POIs and supports filtering by city, tag, affordability, and open/closed status during the requested time window. It defines the system’s “world model” and enables deterministic unit tests using a fixed dataset.

### Module 3: Candidate Retrieval & Heuristic Scoring
**Topics:** Heuristics, utility functions, decision-making  
**Input:** `TripSpec` + POI store  
**Output:** Ranked candidate list of POIs with score breakdown and reason codes  
**Integration:** Feeds the planner with the top-N candidates and explanation metadata  
**Prerequisites:** Module 2  
This module computes a score for each POI using weighted features: interest match, estimated cost, distance penalty (approximate), hours compatibility, and constraint violations. It emits interpretable reason codes (e.g., “matches coffee,” “low cost,” “short travel”) used later for explainable output.

### Module 4: Itinerary Planner (Search Under Constraints)
**Topics:** Search (greedy/beam search), constraint satisfaction  
**Input:** Top-N scored POIs + `TripSpec` + travel-time estimator  
**Output:** Feasible itinerary plan (ordered stops with timestamps and travel segments)  
**Integration:** Produces the core schedule consumed by estimation and reporting  
**Prerequisites:** Module 3  
This module searches over partial itineraries. Each state is a sequence of chosen POIs with accumulated time/cost. The planner expands by adding a next POI and keeps the best partial plans (beam search) according to total score while enforcing constraints: total time ≤ window, POIs open when visited, budget not exceeded, and minimal backtracking.

### Module 5: Time & Budget Estimation + Evaluation Metrics
**Topics:** Evaluation metrics, assessment  
**Input:** Itinerary plan + POI metadata (durations, costs)  
**Output:** Budget/time totals, slack time, feasibility report, constraint check results  
**Integration:** Validates plans and provides measurable outputs for testing and reporting  
**Prerequisites:** Module 4  
This module computes total predicted spending, total visit time, total travel time, and remaining slack. It produces metrics such as “% of budget used,” “minutes of slack,” and a boolean feasibility flag. These metrics make outputs easy to test and compare across planner variants.

### Module 6: Explanation & Command-Line Interface (System Integration)
**Topics:** Agent interaction, explainable AI  
**Input:** User commands + itinerary + score reasons + evaluation metrics  
**Output:** Human-readable itinerary report (text/Markdown) + optional alternatives (“swap” suggestions)  
**Integration:** Connects Modules 1–5 into an end-to-end system a user can run  
**Prerequisites:** Modules 1–5  
This module provides a CLI workflow: load dataset, accept trip request, generate itinerary, and display results. Explanations reference reason codes and metrics (e.g., “chosen because it matches coffee interest and stays under budget”). It can also propose substitutes when constraints change (e.g., lower budget).

---

## Feasibility Study

_A timeline showing that each module's prerequisites align with the course schedule. Verify that you are not planning to implement content before it is taught._

| Module | Required Topic(s) | Topic Covered By | Checkpoint Due |
| ------ | ----------------- | ---------------- | -------------- |
| 1 (TripSpec Parser & Preference Model) | Python basics, data structures, knowledge representation (simple structured objects) | By early course / before CP1 (representation is minimal: dataclasses + validation) | Checkpoint 1 — Wed, Feb 11 |
| 2 (POI Knowledge Base) | Knowledge bases / representation; file I/O; filtering | By early course / before CP1 | Checkpoint 1 — Wed, Feb 11 |
| 3 (Candidate Retrieval & Heuristic Scoring) | Heuristics / utility scoring; basic search over candidates (ranking) | By CP2 (after intro representation; heuristics typically introduced early-mid) | Checkpoint 2 — Thu, Feb 26 |
| 4 (Itinerary Planner: Search Under Constraints) | Search/planning methods (greedy/beam search); constraint satisfaction (time/budget feasibility) | By CP3 (search is a core early-mid topic; planning/search alignment) | Checkpoint 3 — Thu, Mar 19 |
| 5 (Time & Budget Estimation + Evaluation Metrics) | Evaluation metrics; testing & reporting; basic analysis | By CP4 (evaluation is usually available by mid-course; no ML required) | Checkpoint 4 — Thu, Apr 2 |
| 6 (Explanation + CLI Integration) | Agent interaction / explainable output; system integration practices | By CP5 (does not require new AI theory; mostly integration + explanation) | Checkpoint 5 — Thu, Apr 16 |


## Coverage Rationale
Itinera AI covers six AI-relevant areas in a coherent pipeline: (1) knowledge representation (TripSpec + POI schema), (2) knowledge bases (POI dataset/index), (3) heuristic decision-making (scoring and trade-offs), (4) search/planning under constraints (itinerary construction), (5) evaluation metrics (time/budget feasibility and reports), and (6) explainable, agent-facing interaction (reason codes and CLI output). These topics combine naturally to produce intelligent behavior: selecting good options, forming a plan that fits constraints, and justifying decisions to the user.