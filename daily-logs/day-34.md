DAILY LOG — CI/CD PIPELINE MASTERY

Today I continued working toward mastering the complete CI/CD pipeline and focused heavily on understanding the architecture and troubleshooting process.

MAIN PIPELINE:

Build
 ↓
Test
 ↓
Docker
 ↓
GHCR
 ↓
Deploy


1. PIPELINE ARCHITECTURE

I was asked to design the pipeline from memory.

I correctly identified the main dependency chain:

Build → Test → Docker → Deploy

I also understood that each job has a specific responsibility and produces something needed by the next stage.


2. BUILD JOB

The Build job:

- Does not need a previous job.
- Checks out the repository.
- Sets up the required environment.
- Builds/packages the application.
- Creates an artifact.
- Uploads the artifact.

The output is:

flask-build

The artifact is then stored by GitHub Actions so that other jobs can download it.


3. TEST JOB

The Test job:

needs: build

Its responsibilities are:

- Download the build artifact.
- Extract the artifact.
- Install the required dependencies.
- Run pytest.
- Validate the application.

I initially thought the Test job should update and upload another artifact for Docker.

I corrected this understanding.

The same artifact can be downloaded by multiple jobs:

GitHub Artifact Storage
        ↓
   ┌────┴────┐
   ↓         ↓
 Test      Docker

The Test job does not need to create another artifact simply because Docker will also use the build artifact.

The main output of the Test job is:

Tests passed → Docker is allowed to continue.


4. DOCKER JOB

The Docker job:

needs: test

and requires:

permissions:
  packages: write

Its process is:

Download artifact
 ↓
Extract artifact
 ↓
Login to GHCR
 ↓
docker build
 ↓
Tag image using Git SHA
 ↓
docker push
 ↓
GHCR

The Docker image contains the application and the runtime dependencies/resources required according to the Dockerfile.

The important output is the SHA-tagged Docker image stored in GHCR.


5. DEPLOY JOB

The Deploy job:

needs: docker

and requires:

packages: read

Its process is:

Login to GHCR
 ↓
Pull exact SHA-tagged image
 ↓
Deploy

I reinforced that:

docker pull

is not itself the deployment.

Pulling retrieves the image.

The actual deployment requires a deployment platform/environment where the image is run.

Our current project uses an echo command as a simulated deployment rather than a real production deployment.


6. ARTIFACT VS DOCKER IMAGE

I reinforced an important distinction:

Artifact ≠ Docker image

Artifact:

Used mainly as a job-to-job handoff.

Docker image:

A packaged runtime application that can be stored in GHCR and deployed.

The flow is:

Build files
 ↓
Archive
 ↓
GitHub Artifact
 ↓
Test / Docker
 ↓
Docker image
 ↓
GHCR
 ↓
Deploy


7. TROUBLESHOOTING

Today I began practicing CI/CD troubleshooting rather than simply writing pipeline code.

Scenario:

The Test job cannot find requirements.txt.

My first approach was:

1. Check the artifact contents.
2. Check the working directory.
3. Check the runner/environment.

This was mostly correct.

The improved troubleshooting process is:

Read exact error
 ↓
Check artifact
 ↓
Check download
 ↓
Check extraction
 ↓
Check working directory
 ↓
Check file path
 ↓
Check command
 ↓
Check runner/environment


8. PYTEST NOT FOUND

I was given:

pytest: command not found

I correctly identified that the runner needs the required Python/testing environment established before running pytest.

The Test job should explicitly:

- Set up Python 3.12.
- Install dependencies.
- Run pytest.

Example:

python -m pip install -r requirements.txt

Then:

pytest

I learned that we should not rely on undocumented state already existing on the GitHub runner.


9. PYTHON ENVIRONMENT TROUBLESHOOTING

I was given another scenario:

ModuleNotFoundError: No module named 'flask'

while:

pip install -r requirements.txt

reported success.

My initial approach was:

- Check requirements.txt.
- Confirm Flask is included.
- Upgrade pip.
- Reinstall dependencies.

I learned that before reinstalling everything, I should investigate whether there is a Python interpreter/environment mismatch.

Useful checks include:

python --version

python -m pip --version

python -m pip show flask

python -c "import flask; print(flask.__version__)"


10. python -m pip

I learned why this is useful:

Instead of:

pip install -r requirements.txt

we can use:

python -m pip install -r requirements.txt

This makes it clearer that pip belongs to the Python interpreter being used.

This helps identify situations where:

Python A
 ↓
runs the tests

while:

Python B
 ↓
has Flask installed

which can result in:

ModuleNotFoundError


11. IMPORTS

I also remembered that troubleshooting needs to consider whether dependencies are correctly imported and implemented.

For Flask, for example:

from flask import Flask

app = Flask(__name__)

The important debugging principle is:

Read the exact error first.

Different errors point to different layers of the system.


12. DEBUGGING MINDSET

I learned that professional troubleshooting should not immediately jump to reinstalling or changing everything.

The better approach is:

Expected state
 ↓
Actual state
 ↓
Compare
 ↓
Find the first point where they differ
 ↓
Fix the underlying cause


A useful troubleshooting hierarchy is:

1. Does the required file exist?
2. Does requirements.txt contain the dependency?
3. Was the dependency actually installed?
4. Is the correct Python environment being used?
5. Is the dependency imported correctly?
6. Is the application using the dependency correctly?


MAIN TAKEAWAY:

I am becoming better at thinking about CI/CD failures systematically.

Instead of immediately changing commands, I am learning to investigate:

- Artifacts
- File paths
- Working directories
- Runner environments
- Python versions
- Installed packages
- Imports
- Exact error messages

The main professional debugging principle I learned is:

"Diagnose first; fix second."

Another important principle:

"Find the first point where the actual state differs from the expected state."


CURRENT PIPELINE MASTERY:

Architecture understanding     — 🟢 Strong
Job dependencies               — 🟢 Strong
Artifacts                      — 🟢 Strong
Docker                         — 🟢 Strong
GHCR                           — 🟢 Strong
Deploy concepts                — 🟢 Good
Security                       — 🟢 Good
Troubleshooting                — 🟡 Developing
YAML recall                    — 🟡 Developing
Real end-to-end execution      — 🔴 Not completed yet


NEXT SESSION:

Continue the CI/CD troubleshooting exercises.

Start from the:

ModuleNotFoundError: No module named 'flask'

scenario and continue investigating Python environment, dependency installation, imports, and test execution.

Then continue toward the final pipeline mastery challenge:

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

The eventual goal is to implement the complete pipeline independently, run it for real, intentionally troubleshoot failures, and be able to explain every stage and decision.