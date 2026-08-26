
# Teacher Guide



```bash
# Generate a new passcode
echo -n "your-passcode" | sha256sum
# OR
printf '%s' "your-passcode" | sha256sum
```


## Updating the passcode

### PG2CourseEnrollment Repo Changes
1. Generate a new passcode hash
2. Update the `MONTHLY_CODE` variable in `pg2Info.txt` file
3. Update the `PREFIX` variable in `pg2Info.txt` file. This will be the prefix used for the student repos for the upcoming month. EX: `pg2-2607-`
4. Update the `TEMPLATE` variable in `pg2Info.txt` file. Only do this if the template repo has changed for the upcoming month.
   > NOTE: if updating the template repo, then you must add it to the list of selected repos for the bot (don't forget to click the "Save" button)
   > https://github.com/organizations/FullSailGameStudies/settings/installations/143380104 
5. Update the `MAX_REPOS` variable in `pg2Info.txt` file. Count how many students and instructors for the upcoming month and use it for the MAX_REPOS.
6. Commit and push the changes

### FSO Course Changes
Update the Course Materials / Get your GitHub repo assignment with the new passcode used above.


## Listing student repositories

Repositories have a month-specific prefix. Use the GitHub CLI (`gh`) to list them:
> Replace `REPO_PREFIX` with the month prefix you want to clean up (e.g. `pg2-2607-`).
```bash
# List all repos in the org with both topics (table view)
gh repo list FullSailGameStudies --limit 1000 --json name --jq '.[].name' | grep "^REPO_PREFIX" 

# Count total repos for the month
gh repo list FullSailGameStudies --limit 1000 --json name --jq '.[].name' | grep "^REPO_PREFIX" | wc -l 
```

> **Note:** `gh repo list` requires the `repo` scope. Authenticate with `gh auth login` if you haven't already.

## Cloning student repositories

Repositories have a month-specific prefix (e.g. `pg2-2607-`). To clone all repos for a given month, filter the `gh repo list` output by the month prefix and clone each match:


> **NOTE:** replace the `pg2-2608-` with the current month prefix.

```bash
# Preview repos that will be cloned (dry run)
gh repo list FullSailGameStudies --limit 1000 --json name,url --jq '.[] | "\(.name)\t\(.url)"'  | grep "^pg2-2608-" 

# Clone all repos for the month into the current directory
# don't create upstream branch to the forked repo
# each repo is created in its own folder
gh repo list FullSailGameStudies --limit 1000 --json name,url --jq '.[] | "\(.name)\t\(.url)"' | grep '^pg2-2608-' | xargs -L 1 bash -c 'gh repo clone "$2" "$1"' _

```

> **Tip:** If you need to clone into a specific folder, create it first and run the command from inside that folder. Each repo will be cloned into its own subdirectory named after the repo.

## Deleting student repositories

Repositories have a month-specific prefix (e.g. `pg2-2607-`). To delete all repos for a given month, filter by the month prefix.
> Replace `REPO_PREFIX` with the month prefix you want to clean up (e.g. `pg2-2607-`).
```bash
# Preview repos that will be deleted (dry run)
gh repo list FullSailGameStudies --limit 1000 --json name --jq '.[].name' | grep '^REPO_PREFIX'

# Delete all OPS_JUN_Lab* repos
gh repo list FullSailGameStudies  --limit 1000 --json name --jq '.[].name' | grep '^REPO_PREFIX' | xargs -I {} gh repo delete "FullSailGameStudies/{}" --yes
```

> **Warning:** `gh repo delete --yes` skips the confirmation prompt. Always run the dry-run command first to verify which repos will be removed. Deletion is permanent and cannot be undone.

## Deleting closed issues

Student enrollment issues accumulate over time. Use the GitHub CLI to list and delete closed issues in this repository:

```bash
# List closed issues (number and title)
gh issue list --repo FullSailGameStudies/PG2CourseEnrollment --limit 1000 --state closed --json number,title --jq '.[] | "\(.number)\t\(.title)"'

# Count closed issues
gh issue list --repo FullSailGameStudies/PG2CourseEnrollment --limit 1000 --state closed --json number --jq 'length'

# Delete all closed issues
gh issue list --repo FullSailGameStudies/PG2CourseEnrollment --limit 1000 --state closed --json number --jq '.[].number' | xargs -I {} gh issue delete {} --repo FullSailGameStudies/PG2CourseEnrollment --yes
```

> **Warning:** `gh issue delete --yes` skips the confirmation prompt. Always review the list of closed issues before deleting. Deletion is permanent and cannot be undone.
