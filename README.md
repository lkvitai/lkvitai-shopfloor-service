# LKvitai.MES — ShopfloorPilot (Production Tasks & Flow Engine)
**Technical Specification (Draft v0.1)**  
**Project:** LKvitai.MES — ShopfloorPilot  
**Author:** Denis / ChatGPT (for LKvitai.MES)  
**Date:** 2025‑09‑28  
**Status:** Draft (v0.1) — ready for internal review  

---

## 1) Project Overview
We are building a shopfloor execution module that automatically transforms customer orders into production tasks for operators: cut fabric, cut profiles, prepare guides, assemble.  
The focus: **visual configuration of flows** (nodes + dependencies) without coding, **Excel‑like formulas** with IntelliSense, automatic task & label generation, and **real‑time monitoring of bottlenecks** with a WOW effect.  

Guiding principles: configuration‑first, minimum custom code, changeable product types without recompilation, integration with scanners/printers, clear C4/BPMN documentation.  

---

## 2) Objectives (Success)
- Each order automatically splits into a chain of tasks.  
- Operators only see “Take Next” or an auto‑sorted list.  
- Flow balancing: less WIP, assembly never starves.  
- Visual editor for technologists: flows + formulas + hints.  
- Barcode labels for semi‑finished items → full traceability.  
- Real‑time dashboard: **color = delay, thickness = load, animation = flow**.  
- Grafana dashboards: WIP, lead time, bottlenecks.  
- WOW effect: **smart algorithm with coefficients** (KitImpact, DueSoon, SetupGain, Fairness).  

---

## 3) Scope
**In scope (phase‑1)**  
- Master data: task types, work centers, operators, label templates.  
- Rules engine: flows with dependencies, Excel‑like formulas with IntelliSense.  
- Task generation when an order is set to status *Released to Production*.  
- Barcode scanning and semi‑finished label printing.  
- Smart task selection algorithm (“Take Next”) with coefficients.  
- Real‑time monitoring dashboard (nodes + edges with metrics).  
- Metrics export to Prometheus/Grafana.  

**Out of scope (phase‑1)**  
- Advanced AI scheduling (only coefficients now).  
- Full machine integration (only scanners/printers).  
- 3D visualization.  

---

## 4) Constraints & Assumptions
- Tech stack: .NET microservices, MSSQL, frontend on Vue + Vite (Rete.js, Monaco, Tailwind).  
- Single developer (Denis) — maximize reuse, transparent progress (Harvest + GitHub Projects).  
- Deployment: Docker, Portainer, Caddy/Traefik.  
- All rules configurable via UI (no recompilation).  

---

## 5) Actors & Roles
- **Production Manager** — configures flows and formulas, monitors WIP.  
- **Assemblers** — pick tasks for assembly, scan semi‑finished parts.  
- **Machine Operators (Cut, Sew, Profile)** — perform tasks in sequence.  
- **Quality Inspector** — approves/blocks semi‑finished batches.  
- **COO** — reviews KPIs and dashboards.  
- **System Administrator** — manages users, backups, exports.  

---

## 6) Functional Requirements
### 6.1 Task Generation  
- Automatic breakdown of orders into tasks per flow.  
- Excel‑like formulas for dimensions and parameters (with autocomplete).  

### 6.2 Execution & Scanning  
- “Take Next” button → balanced task selection.  
- Label printing and scanning for semi‑finished items.  
- Undo window and full audit log.  

### 6.3 Flow Editor  
- Rete.js visual UI: nodes (operations), arrows (dependencies), formulas inside.  
- Preset flows for standard product types.  

### 6.4 Smart Algorithm (WOW factor)  
Tasks are scored and sorted by coefficients:  
- **KitImpact** — how much a task brings an order closer to assembly (highest weight).  
- **DueSoon** — penalty for approaching due date.  
- **SetupGain** — bonus if current batch matches material/color (fewer changeovers).  
- **Fairness** — correction to prevent cherry‑picking only easy tasks.  
- **WIPGate** — penalize when a work center is overloaded.  

### 6.5 Monitoring  
- Real‑time dashboard:  
  - **Node**: current task, elapsed time, WIP, operators.  
  - **Edge**: queue size, mean wait, throughput.  
- Colors: green/yellow/red by thresholds.  
- Thickness: proportional to throughput/queue.  
- Animated arrows: flow speed = throughput.  

### 6.6 Metrics & Exports  
- Aggregated metrics per work center: WIP, lead time, throughput.  
- Export to Prometheus/Grafana.  

---

## 7) Non‑Functional Requirements
- Availability: 99.5% for shopfloor ops.  
- Performance: task allocation ≤ 200 ms.  
- Auditability: immutable task logs and algorithm coefficients.  
- Security: RBAC, signed events.  
- Deployability: Docker‑ready.  

---

## 8) Data Model (Conceptual)
- **TaskType**: Id, Code, Name, Active.  
- **WorkCenter**: Id, Code, Name, WipLimit.  
- **Operator**: Id, BadgeCode, DisplayName.  
- **Task**: Id, ItemId, WorkCenterId, Status, Priority, KitImpact, DueDate.  
- **TaskLog**: Id, TaskId, EventType, Timestamp, OperatorId.  
- **Label**: Id, TaskId, Type, Template, Status.  
- **Flow**: Id, Code, GraphJson, Active.  
- **MetricDaily**: Date, WorkCenterId, TasksCompleted, AvgLeadTime.  

---

## 9) Services & API
- **Flow Service** — store and fetch flow JSON.  
- **Task Service** — generate tasks, TakeNext algorithm.  
- **Label Service** — generate/print labels.  
- **Metrics Service** — aggregate metrics, expose Prometheus.  
- **Auth/RBAC** — roles, permissions, audit hooks.  

---

## 10) UI / Screens (MVP)
- **Flow Editor** — drag & drop nodes, formulas, presets.  
- **Formula Editor** — Excel‑like editor with autocomplete, colors, error underlines.  
- **Operator UI** — “Take Next”, task list, scan barcode.  
- **Monitoring Dashboard** — flow map with colors/thickness/animation.  
- **Admin UI** — users, roles, label templates.  

---

## 11) User Stories (1 month / 2 sprints)
**Sprint 1 (Weeks 1–2)**  
- As a Production Manager, I can define a flow with nodes and arrows, so that tasks generate automatically.  
- As a Production Manager, I can add formulas with autocomplete, so that task parameters are computed.  
- As a Machine Operator, I can press “Take Next”, so that I always get the right task.  
- As a Machine Operator, I can scan a barcode, so that task status updates instantly.  

**Deliverable:** Flow Editor MVP, Formula Editor MVP, Task Service + TakeNext.  

**Sprint 2 (Weeks 3–4)**  
- As a COO, I can see a live dashboard with colors, thickness, and animation, so that I spot bottlenecks.  
- As a Production Manager, I can view WIP and lead times per work center, so that I plan capacity.  
- As an Operator, I can print/scan semi‑finished labels, so that traceability is preserved.  
- As a System Administrator, I can manage operators and templates, so that system stays consistent.  

**Deliverable:** Monitoring Dashboard MVP, Label printing/scanning, Metrics export to Grafana.  

---

## 12) Risks & Open Questions
- How deep should equipment integration go in phase‑1?  
- Are coefficients (KitImpact, Fairness, DueSoon) enough without full scheduler?  
- Plain barcodes or GS1‑128/QR needed in phase‑1?  
- Do we store flows as JSON in MSSQL only, or also in Git for versioning?  

---

## 13) What developer needs (self‑note)
- MSSQL schema: all tables for tasks, flows, operators, labels, metrics.  
- Vue + Vite frontend project skeleton with Rete + Monaco + Tailwind.  
- Preset JSON flows for Roller, Cassette, Motorized blinds.  
- Coefficient formulas for smart algorithm (KitImpact, DueSoon, SetupGain, Fairness).  
- Prometheus/Grafana dashboards templates.  
- CI/CD pipeline: npm build → wwwroot/app.  
- Seed data: operators, work centers, demo tasks.  
- RBAC roles and test accounts.  

---
