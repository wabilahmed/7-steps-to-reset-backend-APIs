# 7-steps-to-reset-backend-APIs

This repository is being used to rehearse Git workflows for the Backend API project. The containerized environment lets us
create commits locally, but no Git remotes are configured by default. To push changes anywhere (for example to
`https://github.com/wabilahmed/7-steps-to-reset-backend-APIs`), you must add a remote before running `git push`.

## Configure a Remote for Pushing

Follow these steps from the repository root:

1. **Check the current remotes** (optional) to confirm none are configured yet:
   ```bash
   git remote -v
   ```
2. **Add the remote** that should receive the commits. Replace the URL with the remote you want to use if it differs:
   ```bash
   git remote add origin https://github.com/wabilahmed/7-steps-to-reset-backend-APIs.git
   ```
3. **Verify the remote** was registered:
   ```bash
   git remote -v
   ```
   You should now see `origin` pointing to the specified repository for both fetch and push.
4. **Push the current branch** (for example `work`) and set the upstream so future pushes can omit the branch name:
   ```bash
   git push -u origin work
   ```

> ⚠️ You will need valid GitHub credentials or an access token with permission to push to the target repository.

If you ever need to change the remote URL, run `git remote set-url origin <new-url>`. To remove an incorrect remote, use
`git remote remove origin` and then re-add it with the correct URL.

## Dummy File

The `dummy.txt` file simply proves that files can be tracked and committed locally in this environment.
