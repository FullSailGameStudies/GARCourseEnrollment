## 🤖 Organization Workspace Provisioning Engine: Architecture & Lifecycle

This document outlines the end-to-end operational workflow of the automated student repository generation ecosystem. The architecture utilizes static text-file configurations, local C# script generators, and GitHub Actions running on secure organization cloud runners to automatically provision and securely configure private student assignment environments.

------------------------------
## 🏗️ 1. The Core Infrastructure Components
The ecosystem relies on four modular pillars that interact without requiring external persistent database management engines:

* `pg2Info.txt` (The Configuration File): A central configuration file located inside the enrollment repository. It stores structural metadata values (`# PREFIX:`, `# TEMPLATE:`, `# ORG_NAME:`, `# MONTHLY_CODE:`, and `# MAX_REPOS:`) used to drive the downstream Actions pipeline dynamically.
* The Shared Monthly Passcode: A cryptographically randomized 9-character code distributed to students via the LMS. It uses a clean alphanumeric character subset to prevent student entry typos.
* The Enrollment Request Form: A secure, native GitHub Issue Template (`student-invite.yml`) that provides a simplified single-field textbox interface for inputting the passcode.
* The Provisioning Script (`provision2.yml`): A workflow engine powered by an organizational GitHub App integration that runs isolated bash instructions inside on-demand virtual containers.

------------------------------
## 🔄 2. Step-by-Step Data Flow Lifecycle

[Student Submits Form] ➔ [Passcode & Capacity Validated] ➔ [Repo Provisioned] ➔ [PR Created & Issue Closed]

### Phase A: Submission & Authentication

   1. **Student Form Entry:** The student navigates to the private enrollment repository, opens a new workspace request issue ticket, pastes the monthly code phrase, and clicks submit.
   2. **Parallel Triggering:** GitHub immediately spins up an isolated Linux virtual runner. Because the queue restrictions have been dropped, multiple student submissions execute simultaneously in parallel without stepping on file changes.
   3. **Cryptographic Identity Masking:** The runner invokes the pre-built actions/create-github-app-token action block. This helper automatically parses the organization's private cryptographic key (secrets.APP_PRIVATE_KEY) and app ID to generate a short-lived administrative installation access token. By explicitly setting the token's scope to the organization owner (owner: ${{ env.ORG_NAME }}), the generated token is granted permission to cross repository boundaries, making it authorized to access both the private template files and create new repositories.

### Phase B: Verification & Seat Cap Analysis

   1. **Passcode Validation:** The script hashes the passcode submitted by the student using a SHA-256 algorithm and matches it against the expected configuration hash extracted from line 3 of pg2Info.txt. If the passcode does not match, the bot comments an error explanation, closes the ticket as uncompleted, and halts.
   2. **Real-time Capacity Calculation:** The script queries the organization's core repository registry data using a direct gh repo list lookup. It instantly tallies how many active workspaces match your current project prefix string (e.g., pg2-2607-).
   3. **Limit Gatekeeper Protection:** If the live count is equal to or greater than the defined `# MAX_REPOS:` limit, the workflow stops execution, logs a seat capacity limit warning, and closes the ticket to prevent resource leakage or accidental spam.
   4. **Idempotent Duplicate Repository Check:** Before attempting any resource creation, the script uses gh repo view to proactively check if a repository matching the computed target name (Prefix-StudentUsername) already exists in the organization. If a matching workspace is detected, the script identifies it as a duplicate request. It posts a helpful message re-directing the student to their existing private repository, marks the issue as completed, and exits cleanly to prevent overwriting files or breaking existing branches.

### Phase C: Workspace Generation & Collaboration Setup

   1. **Repository Provisioning:** If the safety gates pass, the GitHub App generates from the template repo (e.g. PG2StudentRepo26.7) a brand-new, completely private repository path container under the organization namespace.
   2. **Student Workspace Collaboration:** The script issues a secure PUT command to the `/collaborators` endpoint. Because the task utilizes the scoped GitHub App token rather than a human credential, the invitation is formally branded from the organization entity. It grants individual `Write (Push)` authorization exclusively to that student, keeping their work completely isolated from their classmates.

### Phase D: Feedback Pull Request

   1. **Empty Commits:** To guarantee that the tracking Pull Request can always be opened without a "Nothing to compare" error, the script injects two distinct tracking commits:
	    - It checks out a new branch named feedback, injects an empty initialization commit, and pushes it to the cloud registry.
		- It jumps back to the default master branch, injects a secondary empty initialization commit, and pushes that up as well.
   2. **Grading & Feedback Pull Request:** The script opens a `Grading & Feedback Pull Request` targeting the feedback branch against master. This remains open for the rest of the semester as a dedicated space for instructors to leave reviews, code inline comments, and post grades.

------------------------------
## 🛡️ 3. Built-In Security Engineering Safe-Harbors

* **No Access For Forks:** Because all organization secrets (App IDs and Cryptographic Private Keys) are managed at the core administrative level with strict scope limitations, students who fork the enrollment repository cannot access, read, or execute actions against the department's private grid infrastructure.
* **Zero Token Expirations:** By using an internal GitHub App, the token authentication cycle never expires, ensuring a zero-maintenance automation model across future academic semesters.

------------------------------
