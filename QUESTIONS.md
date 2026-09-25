Questions

### What Was Changed
#### Dockerfile 

The Dockerfile rewrite - build using two Chainguard image.
  Python:latest-dev
    Compile native extensions and install pip dependencies.
  Python:latest
    Distroless runtime — only the app code - is purpose-built to contain only the minimal set of packages required to run Python — no shell, no package manager, no OS utilities

Tool used: https://edu.chainguard.dev/chainguard/containers/migration/migration-tools/dockerfile-conversion/

#### pip.conf 

A new app/pip.conf file was added to redirect all pip install operations to Chainguard's Python Libraries
Primary index: https://libraries.cgr.dev/python-remediated/simple/ — serves CVE-patched versions of Python packages
Fallback index: https://libraries.cgr.dev/python/simple/
The file is passed to the Docker build as a BuildKit secret (--mount=type=secret,id=pip_conf), which means the credentials never appear in any image layer or in docker history.

#### docker-compose.yml 

The compose file was updated to declare the pip_conf secret and pass it to the web service build:
secrets:
  pip_conf:
    file: app/pip.conf


### Why These Changes Were Made
The original image (python:3.11-slim) is a general-purpose Debian-based image. It ships with a large number of OS packages, many of which accumulate CVEs over time and are never relevant to the application's runtime behavior.

### Risks and Issues Reduced

Chainguard's distroless Python image typically reports 0 CVEs in standard scanners (Trivy, Grype) versus dozens in python:3.11-slim.
Chainguard Libraries remediated index serves patched versions of packages without waiting for upstream fixes


### Tradeoffs
Debugging is harder. The distroless runtime image has no shell, no apt, and no common debugging utilities. docker exec -it <container> bash will not work. Debugging must be done by inspecting logs, using the -dev image variant in development, or attaching tools from a sidecar.
Chainguard Libraries requires a subscription token. This introduces an operational dependency: builds will fail if the token expires or network access to libraries.cgr.dev is unavailable. The token must be rotated and managed as a secret.

### What We Would Do Next in a Real Customer Engagement

1. Migrate the Database Service
Replace postgres:15 with cgr.dev/chainguard/postgres to achieve consistent CVE posture across all services, not just the application container.

2. Fix Remaining Secrets
Replace the hardcoded SECRET_KEY with a proper secret injection pattern using environment variable references backed by a secrets manager.
3. Integrate Image Scanning into CI/CD
Add a scanning step to the CI pipeline that:
Fails the build if critical CVEs are introduced
Publishes a before/after CVE count as evidence of improvement
This gives the customer a measurable, auditable security baseline.
4. Set Up Automated Image Update Notifications
Subscribe to Chainguard's image update feed so that when new CVE-patched versions of the base image or remediated packages are published, the team is notified and can cut a new build promptly.

5. Add SBOM Generation
Generate a Software Bill of Materials (SBOM) as part of the build pipeline using Syft or a similar tool.
6. Review and Harden the Full Deployment Stack
Extend the same distroless/non-root/minimal-image pattern to any other services in the stack (reverse proxies, sidecars, init containers).

