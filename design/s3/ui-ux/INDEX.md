# OfferBuddy S3 UI Specification Index

Status: APPROVED / FROZEN

Final Figma Frames: 35
Engineering Specifications: 20
Uncovered Final Frames: 0

---

## Specification Categories

### Focused Apply

| Spec     | Title                       | Surface | Primary Dependencies                                  |
| -------- | --------------------------- | ------- | ----------------------------------------------------- |
| S3-UI-01 | Match Analysis              | Web     | Candidate Profile, Job, Job Intelligence, Preparation |
| S3-UI-02 | Create Tailored Resume      | Web     | Candidate Profile, Match, Preparation, Resume         |
| S3-UI-03 | Base Resume Preview         | Web     | Resume, Preparation                                   |
| S3-UI-04 | Tailored Resume Review      | Web     | Candidate Profile, Job, Preparation, Resume           |
| S3-UI-05 | Focused Cover Letter Review | Web     | Candidate Profile, Job, Match, Resume, Preparation    |

### Candidate Profile

| Spec     | Title                      | Surface | Primary Dependencies                 |
| -------- | -------------------------- | ------- | ------------------------------------ |
| S3-UI-06 | Candidate Profile Overview | Web     | Candidate Profile                    |
| S3-UI-07 | Resume Import Review       | Web     | Candidate Profile, Resume Import, AI |
| S3-UI-08 | Personal Details           | Web     | Candidate Profile                    |
| S3-UI-09 | Professional Summary       | Web     | Candidate Profile                    |
| S3-UI-10 | Skills                     | Web     | Candidate Profile, Resume Import     |
| S3-UI-11 | Experience                 | Web     | Candidate Profile, Resume Import     |
| S3-UI-12 | Education                  | Web     | Candidate Profile, Resume Import     |
| S3-UI-13 | Certifications             | Web     | Candidate Profile, Resume Import     |
| S3-UI-14 | Languages & Eligibility    | Web     | Candidate Profile, Resume Import     |

### Extension / Fast Apply

| Spec     | Title                        | Surface                | Primary Dependencies                                      |
| -------- | ---------------------------- | ---------------------- | --------------------------------------------------------- |
| S3-UI-15 | Extension Floating Assistant | Browser Extension      | Job detection, Application, Preparation, Sponsor Snapshot |
| S3-UI-16 | SEEK Cover Letter Assistance | Extension + Web Config | Default CL, Focused CL, Preparation, SEEK adapter         |

### S2 Integration

| Spec     | Title                          | Surface | Primary Dependencies                                     |
| -------- | ------------------------------ | ------- | -------------------------------------------------------- |
| S3-UI-17 | Application Detail Integration | Web     | Existing S2 Application Detail, Application, Preparation |

### RuoYi Admin

| Spec     | Title                                 | Surface | Primary Dependencies                       |
| -------- | ------------------------------------- | ------- | ------------------------------------------ |
| S3-UI-18 | Sponsor Employers Admin / CRUD        | RuoYi   | Sponsor Dataset, Extension, Admin Security |
| S3-UI-19 | AI Governance / Runtime Configuration | RuoYi   | AI Router, Provider Abstraction, Secrets   |
| S3-UI-20 | AI Monitoring / Operational View      | RuoYi   | AI Observability, Provider Metadata, RBAC  |

---

## Figma Mapping

### S3-01 Focused Apply

| Final Figma Frame                       | Specification |
| --------------------------------------- | ------------- |
| Match Decision                          | S3-UI-01      |
| Create Tailored Resume                  | S3-UI-02      |
| Base Resume Preview                     | S3-UI-03      |
| Tailored Resume Review                  | S3-UI-04      |
| Focused Cover Letter Review             | S3-UI-05      |
| Floating Assistant — Collapsed          | S3-UI-15      |
| Floating Assistant — Hover / Default    | S3-UI-15      |
| Floating Assistant — Sponsor Signal     | S3-UI-15      |
| Floating Assistant — Requirement Review | S3-UI-15      |
| Floating Assistant — Preparation Ready  | S3-UI-15      |
| Floating Assistant — Already Applied    | S3-UI-15      |
| SEEK Cover Letter Step                  | S3-UI-16      |
| SEEK Prepared Cover Letter              | S3-UI-16      |

### S3-02 Candidate Profile

| Final Figma Frame              | Specification |
| ------------------------------ | ------------- |
| Candidate Profile — Main       | S3-UI-06      |
| Candidate Profile — First Time | S3-UI-06      |
| Resume Import Review           | S3-UI-07      |
| Personal Details Edit          | S3-UI-08      |
| Professional Summary Edit      | S3-UI-09      |
| Skills Edit                    | S3-UI-10      |
| Experience Edit                | S3-UI-11      |
| Education Edit                 | S3-UI-12      |
| Certifications Edit            | S3-UI-13      |
| Languages Edit                 | S3-UI-14      |
| Eligibility Edit               | S3-UI-14      |
| Default Cover Letter Edit      | S3-UI-16      |

### S3-03 Application Detail Integration

| Final Figma Frame               | Specification |
| ------------------------------- | ------------- |
| Focused Preparation integration | S3-UI-17      |

### S3-04 RuoYi Admin

| Final Figma Frame              | Specification |
| ------------------------------ | ------------- |
| Sponsor Employers CRUD List    | S3-UI-18      |
| Sponsor Employer Create/Edit   | S3-UI-18      |
| Sponsor Employers Batch Import | S3-UI-18      |
| Sponsor Publish Confirmation   | S3-UI-18      |
| Sponsor Published State        | S3-UI-18      |
| AI Runtime Configuration       | S3-UI-19      |
| AI Capability Edit             | S3-UI-19      |
| AI Monitoring                  | S3-UI-20      |
| AI Failure Detail              | S3-UI-20      |

---

## Core Dependency Graph

Candidate Profile is the factual foundation:

```text
S3-UI-06 Candidate Profile Overview
│
├── S3-UI-07 Resume Import
├── S3-UI-08 Personal Details
├── S3-UI-09 Professional Summary
├── S3-UI-10 Skills
├── S3-UI-11 Experience
├── S3-UI-12 Education
├── S3-UI-13 Certifications
└── S3-UI-14 Languages / Eligibility
        │
        ▼
Candidate Profile + profileVersion
```