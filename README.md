# Solution Engineering Take-Home: Reviewer Queue

This repository contains a small internal tool used to triage customer-facing review items. The goal of this challenge is to make a focused improvement in a realistic support workflow under time pressure.

This is designed for a junior engineer or solution engineer who wants to show they can:

- understand an existing backend or frontend
- fix a real workflow issue
- make a small product experience clearer for a human operator
- think about how the system should communicate with a customer
- explain the issue and fix so a customer can understand clearly

## Timebox

Aim to make one meaningful improvement in about 60 minutes.

Please stop after 90 minutes and ensure your changes are pushed.

You do not need to solve every issue in the app. A small, well-scoped improvement is better than a large incomplete rewrite.

## Scenario

You are supporting a customer operations team that uses this tool to review account requests and decide what to do next.

A reviewer needs to answer three practical questions quickly:

- What should I do with this request?
- Who is handling it right now?
- What should I tell the customer next?

The current app already has the basic queue and workflow actions, but it is missing some clarity around valid transitions and the next communication step.

## Recommended challenge

Pick one of these focused improvements and implement it well:

1. Enforce the workflow rules more clearly on the backend
   - only allow `claim` from `unassigned`
   - only allow `approve`, `reject`, or `escalate` from `in_review`
   - prevent terminal items from taking further action
   - return clear errors for invalid actions

2. Improve the reviewer experience in the UI
   - show the current status and assignee more clearly
   - surface a short, customer-friendly next-step message
   - make it obvious when an action is allowed or blocked

3. Add a simple customer communication hint
   - show a draft message such as "We are reviewing your request" or "We need more information before we can proceed"
   - keep it lightweight and template-based
   - focus on clarity rather than polished copy

## What to submit

Your submission should include:

- a working local app
- one meaningful workflow or UX improvement
- at least one backend or frontend test covering the behavior you changed
- a short `SUBMISSION.md` explaining what you changed and why

## Suggested implementation approach

A strong 60-minute solution could be:

- fix the state transition logic in the API
- add a small message or guidance block in the detail view
- add one targeted test for invalid or terminal-state actions

That gives you a nice mix of backend logic, frontend polish, and customer-facing thinking without requiring a large implementation.

## Setup

### Backend

```bash
cd backend
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
uvicorn app.main:app --reload --port 8000
```

The API runs at `http://localhost:8000`.

### Frontend

In another terminal:

```bash
cd frontend
npm install
npm run dev
```

The frontend runs at `http://localhost:5173`.

## Tests

Backend smoke tests are included:

```bash
cd backend
source .venv/bin/activate
pytest
```

You may add or adjust tests as needed. Focus on behavior that matters to the reviewer workflow.

## Submission notes

You can use [SUBMISSION_TEMPLATE.md](SUBMISSION_TEMPLATE.md) as a starting point for your write-up.

A short walkthrough should ideally cover:

- the workflow problem you chose to fix
- how you handled the backend and frontend changes
- how the app now helps a reviewer communicate with the customer
- how you would communicate this to the customer
