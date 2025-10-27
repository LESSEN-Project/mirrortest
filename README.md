# Tutorial: Sharing Research Output with the LESSEN Organization on GitHub

As part of our project, we would like to make your research output available on our shared GitHub organization page. This way we have a single location where all LESSEN research output is available to project members, industrial partners and the NWO. To achieve this, you have two options:

1. **Transfer the Repository** directly to the LESSEN organization.
2. **Mirror the Repository** using a GitHub Action, which automatically synchronizes your repository with the organization’s repository on each `git push`.

**Please note that your private repositories will remain private even after sharing them on the LESSEN GitHub organization page.** The visibility of each repository can be controlled to ensure that unpublished work stays confidential.

## Pros and Cons of Each Option

### **Option 1: Transfer the Repository**
**Pros**:
   - Simple, one-time setup.
   - No additional maintenance.
   - Centralizes all repository management under the organization.

**Cons**:
   - Full ownership is transferred to the organization, limiting control.
   - Access management requires organization admin involvement.

### **Option 2: Mirror the Repository**
**Pros**:
   - Maintains independent ownership of the original repository.
   - Automated synchronization on every `git push`.
   - Retains flexibility for ongoing development.

**Cons**:
   - Requires initial setup of SSH keys and GitHub Action configuration.
   - Potential for configuration errors if SSH setup isn’t correct.
   - Regular monitoring may be needed to ensure mirroring works as expected.

---

## Option 1: Transfer the Repository

1. **Go to Your Repository**: Navigate to your repository on GitHub.
2. **Settings**: In the upper-right corner of the repository, click on **Settings**.
3. **Transfer Ownership**:
   - Scroll down to the **Danger Zone** section.
   - Click **Transfer** to begin transferring ownership.
4. **Enter Organization and Repository Name**:
   - Specify the organization name as `LESSEN-Project`.
   - Confirm by typing in the repository’s name and completing the process.
   
Once transferred, the repository will appear under the LESSEN organization, and you will retain contributor access. **If your repository is private, it will remain private in the organization, ensuring that your unpublished work is accessible only to you and authorized team members.**

---

## Option 2: Mirror the Repository with GitHub Actions

To use the mirroring option, you need to set up a GitHub Action that pushes any updates from your repository to the LESSEN organization’s copy of your repository.

### Step 1: Create an Empty Repository on the LESSEN Organization Page

1. **Go to the LESSEN GitHub Page**: Navigate to `https://github.com/LESSEN-Project`.
2. **Create a New Repository**:
   - Click **New** to create a new repository.
   - Name the new repository exactly as your local repository name.
   - **If necessary**, set the repository to **private** to keep your work secure.
   - **Do not** initialize the repository with a README or other files.

### Step 2: Create a GitHub Action in Your Repository

1. **Add a Workflow File**:
   - In your repository, navigate to `.github/workflows/`. Create these directories if they do not exist.
   - Create a file named `mirror.yml` and paste the following code:

     ```yaml
      # GitHub Action: Mirror Repository to LESSEN Organization
      # This action automatically pushes updates from this repository to the LESSEN-Project GitHub organization
      # repository whenever changes are pushed.
      
      name: Mirror Repository to LESSEN Organization
      on: push

      jobs:
        mirror:
          # Prevent the action from running if the push event originates from the LESSEN organization, to avoid recursive looping
          if: ${{ github.repository_owner != 'LESSEN-Project' }}
      
          runs-on: ubuntu-latest
      
          steps:
            # Step 1: Check out the repository with full history
            - name: Checkout the repository
              uses: actions/checkout@v4
              with:
                fetch-depth: 0  # Ensures full commit history is fetched, necessary for mirroring all branches
      
            # Step 2: Set up SSH to authenticate with the LESSEN organization
            - name: Set up SSH for GitHub
              env:
                MIRROR_REPO_SSH_KEY: ${{ secrets.MIRROR_REPO_SSH_KEY }}  # Uses the private SSH key stored as a secret
              run: |
                mkdir -p ~/.ssh  # Create SSH directory
                echo "$MIRROR_REPO_SSH_KEY" > ~/.ssh/mirror_repo_key  # Save SSH key to file
                chmod 600 ~/.ssh/mirror_repo_key  # Set secure permissions for the key
                ssh-keyscan github.com >> ~/.ssh/known_hosts  # Add GitHub to known hosts for secure SSH
                git config core.sshCommand "ssh -i ~/.ssh/mirror_repo_key"  # Configure git to use this SSH key
      
            # Step 3: Add the LESSEN organization repository as a remote and push changes
            - name: Push to mirror repository
              run: |
                # Add the LESSEN organization repository as a new remote called "mirror"
                git remote add mirror git@github.com:LESSEN-Project/<repo_name>.git
                
                # Force push all branches and tags to the mirror repository
                git push --force --all mirror
                git push --force --tags mirror

     ```

2. **Replace `<repo_name>`**: Update `<repo_name>` in the `mirror.yml` file with the actual name of your repository.

### Step 3: Set Up SSH Key Authentication

1. **Generate an SSH Key** (if you don’t have one already):
   - On your machine, run:
     ```bash
     ssh-keygen -t ed25519 -C "your_email@example.com"
     ```
   - This command generates a public and private key pair. The keys are saved to the `~/.ssh` directory.

2. **Add SSH Public Key to GitHub profile**:
   - Copy the public key (usually in `~/.ssh/id_ed25519.pub`).
   - Go to **Settings** on your GitHub profile.
   - Under **SSH and GPG keys**, add the new SSH key.

3. **Add SSH Private Key to Repository Secrets**:
   - Copy the contents of the private key (e.g., `~/.ssh/id_ed25519`).
   - In your repository on GitHub, go to **Settings > Secrets and variables > Actions**.
   - Add a new secret named `MIRROR_REPO_SSH_KEY`, and paste the private key as the value.

### How to Display SSH Keys

To make sure you’ve copied the correct keys, you can display them in the terminal:

- **Display Private Key** (for GitHub Secret setup):
  ```bash
  cat ~/.ssh/id_ed25519
  ```
  *Copy the output carefully and paste it as the value for `MIRROR_REPO_SSH_KEY` in your GitHub repository’s secrets.*

- **Display Public Key** (for GitHub Profile setup):
  ```bash
  cat ~/.ssh/id_ed25519.pub
  ```
  *Copy the entire output and add it to **Settings > SSH and GPG keys** on your GitHub profile.*

Once set up, this GitHub Action will push all branches and tags to the LESSEN organization’s copy whenever you push changes to your repository.

---

## 🧩 Troubleshooting

### Problem: GitHub Action fails with “Permission denied (publickey)” or similar SSH/HTTPS errors

This usually happens if your repository is using **HTTPS** instead of **SSH**, or if your SSH key has not been added correctly to GitHub or your repository secrets.

### How to Fix the Issue by Switching to SSH

1. **Check your current remote:**

   * Run the following command in your local repository:

     ```bash
     git remote -v
     ```
   * If you see something like this:

     ```
     origin  https://github.com/<user>/<repo>.git (fetch)
     origin  https://github.com/<user>/<repo>.git (push)
     ```

     your repository is currently using **HTTPS**.

2. **Change the remote to SSH:**

   * Replace `<user>` and `<repo>` with your actual GitHub username and repository name:

     ```bash
     git remote set-url origin git@github.com:<user>/<repo>.git
     ```
   * Verify the change:

     ```bash
     git remote -v
     ```

     You should now see:

     ```
     origin  git@github.com:<user>/<repo>.git (fetch)
     origin  git@github.com:<user>/<repo>.git (push)
     ```

3. **Test your SSH connection:**

   * Run:

     ```bash
     ssh -T git@github.com
     ```
   * You should see a message like:

     ```
     Hi <username>! You've successfully authenticated.
     ```

4. **(Optional) Use SSH by default for all GitHub repositories:**

   * To avoid future HTTPS issues, you can configure Git to always use SSH for GitHub:

     ```bash
     git config --global url."git@github.com:".insteadof https://github.com/
     ```
   * To undo this later, run:

     ```bash
     git config --global --unset url."git@github.com:".insteadof
     ```

After switching to SSH, re-run your GitHub Action — it should now be able to push to the LESSEN organization repository successfully.
