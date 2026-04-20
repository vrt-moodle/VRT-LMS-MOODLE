# VRT-LMS-MOODLE

Repository bootstrap for VRT LMS based on Moodle, prepared for TA review and developer onboarding.

## Repository structure

- `vrt-lms-moodle/`: Moodle application source code.
- `docker/`: Local and test Docker runtime (`Dockerfile`, `docker-compose.yml`).
- `DEPLOY-CHECKLIST.md`: Deployment acceptance checklist for test environment.
- `Moodle-Test-Deployment-Runbook.md`: End-to-end setup and deployment runbook.
- `SECURITY-ROTATION-CHECKLIST.md`: Secret rotation and security handover checklist.

## Prerequisites

- Docker Engine + Docker Compose plugin.
- Git.
- Access to PostgreSQL server for test environment.

## Quick start (local/test)

1. Create runtime data folder:
   - `mkdir moodledata`
2. Prepare app config from template:
   - Copy `vrt-lms-moodle/config.php.example` to `vrt-lms-moodle/config.php`
   - Fill real DB connection and `wwwroot`
3. Build and start services:
   - `cd docker`
   - `docker compose up -d --build`
4. Verify:
   - `docker compose ps`
   - Open `http://localhost:5090`

## Security baseline

- Do not commit runtime or secret files (`config.php`, `.env`, `moodledata`, DB dumps).
- Use dedicated DB user for app (for example `moodle_app`), avoid privileged users.
- If credentials are exposed in any channel, rotate immediately and follow `SECURITY-ROTATION-CHECKLIST.md`.

## TA review checklist

- Confirm branch is clean and no sensitive files are tracked.
- Verify Docker services can start with documented steps.
- Validate test acceptance items in `DEPLOY-CHECKLIST.md`.
- Review PR against security and operational checklists.

## Team conventions

- Branch naming:
  - `feature/<ticket-or-scope>`
  - `fix/<ticket-or-scope>`
  - `chore/<scope>`
- Prefer small, reviewable pull requests.
- Use the pull request template and keep test notes explicit.