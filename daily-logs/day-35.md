DAILY LOG — CI/CD PIPELINE MASTERY

Today I moved from learning the pipeline conceptually to actually running the project through GitHub Actions.

MAIN PIPELINE:

Build
 ↓
Artifact
 ↓
Test
 ↓
Docker
 ↓
GHCR
 ↓
Deploy


1. PROJECT SETUP

I created the Flask CI/CD project locally in WSL and connected it to GitHub.

The repository is using:

- Git
- GitHub
- Python
- Flask
- pytest
- Docker
- GitHub Actions
- GHCR


2. GIT REPOSITORY

I initialized the project as a Git repository.

I configured the GitHub remote and renamed the branch to:

main

I initially received:

error: src refspec main does not match any

I learned that this happened because the local repository did not have a commit yet.

The solution was:

git add .
git commit -m "Initial Flask CI/CD project"
git push -u origin main

This successfully established the repository and pushed the project to GitHub.


3. GITHUB ACTIONS WORKFLOW

I created:

.github/workflows/ci-cd.yml

The workflow contains four jobs:

Build
 ↓
Test
 ↓
Docker
 ↓
Deploy

The jobs use:

needs: build
needs: test
needs: docker

to control their execution order.


4. BUILD JOB

The Build job:

- Checks out the repository.
- Sets up Python.
- Installs dependencies.
- Creates a tar.gz archive.
- Uploads the archive as a GitHub Actions artifact.

Artifact:

flask-build

The artifact is used as the handoff between CI jobs.


5. TEST JOB

The Test job:

needs: build

It:

- Downloads the build artifact.
- Extracts it.
- Installs the dependencies.
- Runs pytest.

I reinforced the difference between an artifact and a Docker image.

Artifact:

Used for transferring build files between GitHub Actions jobs.

Docker image:

Used to package the application for container execution and deployment.


6. DOCKER JOB

The Docker job:

needs: test

Permissions:

packages: write

It:

- Downloads the build artifact.
- Extracts it.
- Logs into GHCR.
- Builds the Docker image.
- Tags the image using the Git SHA.
- Pushes the image to GHCR.


7. REAL PIPELINE FAILURE

The Docker job initially failed.

The error was:

repository name must be lowercase

The workflow was using:

ghcr.io/${{ github.repository }}:${{ github.sha }}

The GitHub repository name contained an uppercase character:

RaghavN6/flask-app

Docker requires the repository/image name to be lowercase.

I learned that GitHub's repository context can contain uppercase characters and that Docker image names must follow Docker's naming requirements.


8. FIXING THE DOCKER IMAGE NAME

The Docker image name was changed to use the lowercase GitHub repository value:

IMAGE_NAME="ghcr.io/${GITHUB_REPOSITORY,,}:${GITHUB_SHA}"

This converts:

RaghavN6/flask-app

into:

raghavn6/flask-app

The resulting image becomes:

ghcr.io/raghavn6/flask-app:<SHA>

This is a valid Docker image name.


9. DEPLOY JOB

I also applied the same lowercase image-name correction to the Deploy job.

Deploy:

needs: docker

Permissions:

packages: read

It:

- Logs into GHCR.
- Pulls the SHA-tagged image.
- Performs the deployment step.

The current deployment is simulated with echo.

The actual infrastructure deployment will be added later.


10. GITHUB COMMITS

I pushed the updated workflow to GitHub.

The latest commit is:

Update ci-cd.yml

This confirms that the corrected workflow is now present on the main branch.

The previous failed workflow runs remain visible in GitHub, but they represent previous versions of the workflow and do not mean the latest fix failed.


11. TROUBLESHOOTING LESSON

Today's biggest practical lesson was troubleshooting a real CI/CD failure.

The process was:

Pipeline
 ↓
Docker job
 ↓
Read exact error
 ↓
Identify invalid image tag
 ↓
Inspect repository name
 ↓
Identify uppercase character
 ↓
Correct image naming
 ↓
Commit fix
 ↓
Push new workflow
 ↓
Run pipeline again


12. IMPORTANT DEVOPS PRINCIPLE

I practiced:

"Read the exact error before changing anything."

The pipeline had already successfully completed:

- Set up job
- Download artifact
- Extract artifact
- Login to GHCR

The failure was specifically at:

Docker build

This allowed the problem to be narrowed to Docker image naming instead of unnecessarily changing the entire pipeline.


13. CURRENT PROJECT STATUS

Git repository              — ✅
GitHub repository           — ✅
GitHub Actions workflow     — ✅
Build                       — ✅
Artifact                    — ✅
Test                        — ✅
Docker artifact extraction  — ✅
GHCR login                  — ✅
Docker image naming         — 🔧 Fixed
Docker build                — ⏳ Verify next run
GHCR push                   — ⏳ Verify
Deploy                      — ⏳ Verify


14. NEXT SESSION

Tomorrow I will check the newest GitHub Actions run after the image-name fix.

Expected flow:

Build
 ↓
Test
 ↓
Docker Build
 ↓
Push to GHCR
 ↓
Deploy

If another stage fails, I will inspect the exact error and troubleshoot that stage rather than changing unrelated parts of the pipeline.

The final goal is:

Build → Test → Docker → GHCR → Deploy

with the entire pipeline successfully executing end-to-end.


MAIN TAKEAWAY:

Today I moved from writing a theoretical CI/CD pipeline to actually running and debugging one.

I encountered a real Docker/GHCR naming failure, identified the exact cause, corrected the workflow, committed the fix, and pushed it to GitHub.

This is an important step toward practical CI/CD mastery because I am now learning how to diagnose and fix real pipeline failures rather than only studying successful examples.