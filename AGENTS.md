# AGENTS.md

Instructions for the coding agents and the reviewer working on this repository.

## What we are building

**Appointment Book (Zeszyt wizyt)**: a minimalist, modern, easy-to-use web app (PWA) for small service providers (hairdresser, beautician, tire shop) who still keep a paper calendar.

- Appointments live in the provider's **Google Calendar**. The app has no database of its own.
- When the provider books a client, the app checks whether the slot is free and asks "Slot free, book it?".
- The day before the visit, the client gets a personalized reminder SMS.
- The provider needs a Google account; the client does not need any account.
- Later: a photo of a paper calendar becomes a draft list of appointments that a human approves.

Status: the first working slice is being built (free-slot check, booking, fake SMS). Commands below describe the planned stack and become valid when that slice is merged.

## Setup and verification

- Stack: Python 3.11+, FastAPI, plain HTML/CSS/JS, pytest.
- Setup: `python -m venv .venv && . .venv/bin/activate && pip install -r requirements.txt`
- Run: `uvicorn app.main:app` and open http://127.0.0.1:8000
- Verify: `pytest` must pass before every PR. Tests run on the fake calendar and need no network and no Google account.
- Done means: every acceptance criterion in the ticket is checked by running the behavior, including at least one edge or failure case. A clean review does not prove the ticket is complete.

## Project rules

### 1. Use only fictional data
- **Applies to:** the whole repository: code, tests, fixtures, README, screenshots.
- **Why:** the repository is public and appointments contain client names and phone numbers, which are personal data (GDPR). Use made-up people such as "Jan Testowy, +48 500 000 000".

### 2. Reach the calendar only through the `CalendarProvider` interface
- **Applies to:** every piece of code that reads or writes appointments. Interface: list appointments, add appointment, is slot free.
- **Why:** the agents have no access to a real Google account, so tests must run on a fake in-memory calendar. The Google Calendar adapter is a swappable second implementation. Never call the Google API from routes or UI code directly.

### 3. Keep secrets and tokens out of the repository
- **Applies to:** all files and all commits. Read Google OAuth tokens and SMS provider keys from environment variables. `.env`, token files and `*.db` stay in `.gitignore`.
- **Why:** the repository is public. Google OAuth tokens in testing mode expire after 7 days, so renewing access is a manual step for the owner and must never be "fixed" by committing a token.

### 4. Send SMS only through the fake sender
- **Applies to:** the `app/sms` module. The default sender writes the message to the log and sends nothing. A real provider needs an explicit ticket from the owner.
- **Why:** real SMS cost money and reach real people. The reminder text is a template with variables (for example salon name and time) so the owner can personalize it.

### 5. One ticket, one branch, one small PR
- **Applies to:** all work. Branch from the ticket, put the ticket link and its acceptance criteria in the PR description, and keep the diff focused on that ticket. Never merge your own PR; the owner decides.
- **Why:** Qodo review and human review work best on small, focused changes, and each ticket stays traceable to the behavior it promised.
