# New Application Page Specification

## Overview

| Item | Value |
|---|---|
| Page | New Application |
| Sprint | Sprint 1 |
| Status | Design Approved |
| Primary Purpose | Create a new job application |
| Primary User | Active job seeker |

---

## Purpose

The New Application page allows users to create a new application with the minimum amount of manual work.

OfferBuddy prioritises AI-assisted job parsing while always providing a manual entry option.

---

## User Question

The page should answer:

> How can I quickly create a new application?

---

# Design Artifact

**Figma**

https://www.figma.com/design/9otw4DIbkS1J5nG0stMkuW/application?node-id=8-2&t=dDBpAm3B1eeNLPUI-0

---

## User Flow

### AI-assisted Flow (Primary)

```text
Paste Job Link
        ↓
Parse Job
        ↓
Review Parsed Information
        ↓
Edit if Required
        ↓
Save Application
```

### Manual Flow (Fallback)

```text
New Application
        ↓
Continue Manually
        ↓
Complete Form
        ↓
Save Application
```

---

## Primary Goal

Enable users to create an application as quickly as possible.

Manual typing should be minimised whenever possible.

---

## Primary Action

### Parse Job

Users paste a SEEK, LinkedIn, Indeed, or company job URL.

OfferBuddy extracts available information automatically.

---

## Secondary Action

### Continue Manually

Users can bypass AI parsing and complete the form manually.

Typical scenarios include:

- AI parsing failed
- Internal company careers page
- Historical application
- No job URL available

---

# Page States

## State 1

Initial

Displays:

- Job URL input
- Parse Job button
- Continue Manually option

---

## State 2

Parsed

Displays:

- Parsed application information
- Editable form
- Success message

Example:

```text
Job information extracted successfully.

Please review before saving.
```

---

## State 3

Manual

Displays an empty editable application form.

---

# Application Fields

## Required

- Company
- Position
- Applied Date
- Status

---

## Recommended

- Location
- Work Arrangement
- Salary
- Visa Sponsorship
- Source
- Job Link

---

## Optional

- Notes

---

# Field Behaviour

All parsed fields remain editable.

Users may modify any value before saving.

No field should become read-only after AI parsing.

---

# Validation

Required fields:

- Company
- Position
- Applied Date
- Status

The Save button remains disabled until required fields are completed.

---

# Success Behaviour

After saving successfully:

- Application is persisted.
- User returns to the Applications page.
- The newly created application appears at the top of the list.

---

# Error Behaviour

When parsing fails:

- Display a clear error message.
- Preserve the pasted URL.
- Allow Retry.
- Allow Continue Manually.

Example:

```text
We couldn't extract all job details.

You can retry or continue by entering the information manually.
```

---

# Out of Scope

Sprint 1 excludes:

- Multiple job links
- Resume upload
- Cover letter generation
- AI resume optimisation
- Interview preparation
- Automatic application submission

---

# Design Principles

## AI First

AI parsing is the preferred workflow.

---

## Manual Always Available

Users must always have the option to enter information manually.

---

## Minimal Friction

Only request information required to manage an application.

---

## Editable AI

AI-generated information should always be editable before saving.

---

## Accessibility

- Keyboard accessible
- Clear labels
- Required fields identified
- Validation messages understandable
- Colour is not the only indicator of errors

---

## Responsive Behaviour

### Desktop

Single centred form.

### Mobile

Fields stack vertically.

Buttons span available width.

---

# Success Criteria

The page is successful when:

- Users can create an application from a job URL.
- Users can create an application manually.
- AI parsed information can be reviewed before saving.
- Required validation prevents incomplete records.
- Creating an application feels significantly faster than using Excel.