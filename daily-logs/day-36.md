DAILY LOG — CI/CD PIPELINE MASTERY

Today I moved from conceptual CI/CD learning into the practical implementation phase.

MAIN OBJECTIVE:

Begin building and validating the real Flask CI/CD project rather than only working with rough pipeline templates.

CURRENT MAIN PIPELINE:

Git Push
 ↓
BUILD
 ↓
TEST
 ↓
DOCKER
 ↓
GHCR
 ↓
DEPLOY


1. LOCAL FLASK APPLICATION

I created the actual Flask application and validated it locally.

The application contains:

GET /
    ↓
Returns application message

GET /health
    ↓
Returns healthy status

The health endpoint will later be useful for deployment verification and monitoring.


2. PYTHON DEPENDENCIES

Created:

requirements.txt

The dependencies are version-pinned:

Flask==3.1.1
pytest==8.4.1

I learned that pinning versions improves reproducibility because the pipeline can use predictable dependency versions.


3. PYTEST

Created:

tests/test_app.py

The tests validate:

- Main application endpoint
- Health endpoint
- HTTP status codes
- Returned JSON

The tests were successfully executed locally.

Result:

2 passed


4. DOCKER

Created a Dockerfile for the Flask application.

The Dockerfile:

- Uses Python 3.12 slim.
- Sets `/app` as the working directory.
- Copies requirements.txt.
- Installs dependencies.
- Copies app.py.
- Exposes port 5000.
- Starts the Flask application.

The Docker image was successfully built locally.


5. LOCAL CONTAINER

Successfully ran the Docker container locally.

The application was tested through:

GET http://localhost:5000/

and:

GET http://localhost:5000/health

Both worked correctly.

This means the local application lifecycle has been validated:

Flask
 ↓
pytest
 ↓
Docker build
 ↓
Docker container
 ↓
Application


6. GIT

The project was already connected to the GitHub repository.

Verified:

- Branch: main
- Remote: origin/main
- Working tree: clean
- Repository synchronized with GitHub

The `.gitignore` was also configured to prevent unnecessary files such as:

.venv/
__pycache__/
.pytest_cache/
*.pyc

from being committed.


7. PRACTICAL PROJECT STRUCTURE

Current project structure:

flask-cicd/
├── app.py
├── requirements.txt
├── Dockerfile
├── tests/
│   └── test_app.py
└── .github/
    └── workflows/
        └── ci-cd.yml


8. CI/CD WORKFLOW DESIGN

I designed the overall workflow from memory.

The main flow is:

BUILD
 ↓
TEST
 ↓
DOCKER
 ↓
DEPLOY


BUILD:

Responsibilities:

- Prepare application.
- Prepare dependencies.
- Package/archive required files.
- Upload the archive as a GitHub Actions artifact.

Output:

flask-build artifact.


TEST:

Needs:

needs: build

Responsibilities:

- Download the build artifact.
- Extract it.
- Install required dependencies.
- Run pytest.
- Validate the application.

The successful Test job allows Docker to continue.


DOCKER:

Needs:

needs: test

Permissions:

packages: write

Responsibilities:

- Download the artifact.
- Extract the application.
- Login to GHCR.
- Build the Docker image.
- Tag the image using the Git SHA.
- Push the image to GHCR.

Output:

SHA-tagged Docker image stored in GHCR.


DEPLOY:

Needs:

needs: docker

Permissions:

packages: read

Responsibilities:

- Login to GHCR.
- Pull the exact SHA-tagged Docker image.
- Deploy/run the container.

Important correction learned:

Deploy does NOT download the GitHub Actions artifact.

Instead:

Build artifact
 ↓
Test / Docker

Then:

Docker image
 ↓
GHCR
 ↓
Deploy

Therefore:

Artifact → CI job handoff

Docker image → deployment artifact/runtime package


9. GHCR UNDERSTANDING

The Docker job needs:

packages: write

because it needs to push the Docker image.

The Deploy job needs:

packages: read

because it only needs to pull the image.

The image should be referenced using:

ghcr.io/${{ github.repository }}:${{ github.sha }}

This gives every image a traceable version tied to a specific Git commit.


10. PIPELINE ARCHITECTURE UNDERSTANDING

I can now explain the purpose of each stage:

Build:

Prepare and package the application.

Test:

Validate that the application works.

Docker:

Package the validated application into a container image.

GHCR:

Persist and distribute the Docker image after the temporary GitHub runner disappears.

Deploy:

Retrieve the exact image from GHCR and deploy it.


11. IMPORTANT PIPELINE PRINCIPLE

The GitHub runner is temporary.

Therefore:

Docker image built on runner
 ↓
Runner finishes
 ↓
Image disappears with runner

This is why the image must be pushed to a registry such as GHCR.

Then another runner or deployment environment can:

Login
 ↓
Pull image
 ↓
Deploy


12. CURRENT PRACTICAL PROGRESS

Local Flask application       ✅
Local pytest tests             ✅
Dockerfile                     ✅
Docker image build             ✅
Docker container               ✅
Application health endpoint    ✅
Git repository                 ✅
GitHub repository              ✅
Workflow directory             ✅
Pipeline architecture          ✅
Complete CI/CD YAML            🟡 Not yet completed
GitHub Actions execution       🔴 Not yet run
GHCR push                      🔴 Not yet run
Real deployment                🔴 Not yet implemented


13. CURRENT MASTERY STATUS

CI/CD concepts                 🟢 Strong
GitHub Actions concepts        🟢 Strong
Artifacts                      🟢 Strong
Docker                         🟢 Strong
GHCR                            🟢 Strong
Secrets & permissions           🟢 Good
Deployment concepts             🟢 Good
Troubleshooting                 🟡 Developing
Practical implementation        🟡 In progress
End-to-end pipeline             🔴 Next milestone


MAIN TAKEAWAY:

I have now moved from understanding the pipeline conceptually to actually building the application that the pipeline will operate on.

The local application has been proven to work.

The next step is to write the complete GitHub Actions workflow and execute it for real.


NEXT SESSION:

Finish:

.github/workflows/ci-cd.yml

The complete workflow should implement:

Git Push
 ↓
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

I will write the first complete version myself and use my previous daily logs as a reference if necessary.

Then the workflow will be:

1. Reviewed.
2. Corrected.
3. Committed.
4. Pushed to GitHub.
5. Executed by GitHub Actions.
6. Debugged if any jobs fail.

The goal is not merely to make the YAML syntactically correct.

The goal is to successfully execute the complete pipeline end-to-end.