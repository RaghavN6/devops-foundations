DAILY LOG — CI/CD PROJECT

Today I returned to my CI/CD learning after taking a week-long break to rest and refresh my mind.

I continued the Flask CI/CD project and focused on completing the Docker → GHCR → Deploy stages.

WHAT I LEARNED:

1. Docker Job

I reinforced that the Docker job should only run after the Test job succeeds:

needs: test

The Docker job runs on a fresh runner, so it needs to download the artifact produced by the previous job and extract it before building the Docker image.

The flow is:

Test
→ Download artifact
→ Extract artifact
→ Login to GHCR
→ Build Docker image
→ Push image to GHCR


2. GHCR

I learned how GHCR (GitHub Container Registry) is used to store Docker images so they can be accessed by other runners or deployment environments.

The basic process is:

docker build
→ Docker image
→ docker push
→ GHCR
→ docker pull
→ Deployment environment


3. Authentication and Permissions

I reinforced the difference between authentication and authorization.

Authentication determines who is accessing GHCR.

Authorization determines what that identity is allowed to do.

For the Docker job:

packages: write

This allows the job to push the Docker image.

For the Deploy job:

packages: read

This allows the job to pull the Docker image.

This follows the principle of least privilege.


4. Git SHA Image Tagging

I learned why using the Git SHA as the Docker image tag is better for traceability than relying only on "latest".

Example:

ghcr.io/${{ github.repository }}:${{ github.sha }}

This connects the Docker image directly to the Git commit that produced it.

Therefore the same exact image can be pushed to GHCR and later pulled during deployment.


5. Deploy Job

I independently reconstructed the Deploy job.

The process is:

Docker job succeeds
→ Fresh Deploy runner
→ Login to GHCR
→ Pull exact Docker image
→ Deploy

The Deploy job uses:

needs: docker

and only requires:

packages: read


6. Deployment

The current deployment step is only a simulation using:

echo "Deploying ..."

It does not yet deploy to a real production environment.

A real deployment could later use AWS EC2, ECS, EKS, Kubernetes, etc.


PRACTICAL WORK:

I independently wrote the Docker job and Deploy job from memory after taking a week away from studying.

I was able to remember the overall architecture correctly.

My main mistakes were YAML formatting and GitHub Actions expression syntax rather than misunderstanding the concepts.

For example:

Incorrect:
{{ github.actor }}

Correct:
${{ github.actor }}


CURRENT PROJECT:

Build       — Complete
Test        — Complete
Docker      — Mostly complete
GHCR        — Implementation prepared
Deploy      — Implementation prepared
End-to-end  — Not yet tested


REMAINING WORK:

The Build job still needs to properly create the artifact expected by the Test job.

Required artifact:

dist/flask-app.tar.gz

Artifact name:

flask-build

After that I need to:

1. Complete artifact creation in Build.
2. Verify Test can download and extract it.
3. Assemble the complete workflow.
4. Validate the YAML.
5. Push the workflow to GitHub.
6. Run the pipeline.
7. Debug any failures.
8. Verify the Docker image is pushed to GHCR.
9. Verify Deploy can pull the exact image.
10. Complete the end-to-end pipeline.


MAIN TAKEAWAY:

After taking a week away from studying, I was able to recall most of the Docker, GHCR, and Deploy architecture.

This shows that the concepts are starting to stick.

My main weakness is still remembering exact YAML syntax and implementing the complete pipeline without assistance.

My conceptual understanding is stronger than my syntax recall.

The goal is to move from:

"I understand it when I see it."

to:

"I can build it from memory."

and eventually:

"I can build it, troubleshoot it, and explain it to someone else."


NEXT SESSION:

Continue with creating the Build artifact:

Build
→ Create dist/flask-app.tar.gz
→ Upload as flask-build

Then finish and run the complete:

Build → Test → Docker → GHCR → Deploy