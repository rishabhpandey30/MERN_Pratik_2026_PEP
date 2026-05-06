 📦 Sonatype Nexus: The DevOps Artifact Warehouse
--------------------------------------------------------------------------
Q. What is Sonatype Nexus?

Sonatype Nexus is an **Artifact Repository Manager**.
In a professional CI/CD pipeline, we don't just move code; we move "Artifacts" (compiled, packaged, and versioned files).
Nexus serves as the central hub where these artifacts are stored, organized, and retrieved.
---------------------------------------------------------------------------
Q. Why do we need Nexus? 

**Version Control for Builds**: Just as GitHub tracks code changes, Nexus tracks build changes.
It allows you to store multiple versions (e.g., `1.0.1`, `1.0.2`) of the same application.
**Rollback Capability**: If Build #10 fails on the live site, you can instantly pull the "Golden Copy" of Build #9 from Nexus and re-deploy it.
**Single Source of Truth**: It ensures that the exact same file that was tested in Jenkins is the one that gets deployed to production.
**Resource Management**: By offloading storage to Nexus, you can keep your Jenkins workspace clean using `cleanWs()` without losing your build history.
---------------------------------------------------------------------------
Q.How Nexus Works in our Pipeline ?

1. **Build**: Jenkins pulls code from GitHub and builds a Docker image.
2. **Package**: The project is compressed into a `.tar` file (e.g., `frontend-app-1.0.7.tar`).
3. **Upload**: Using the `nexusArtifactUploader` plugin and the `nexus-creds` stored in Jenkins, the file is pushed to a **Raw (Hosted)** repository.
4. **Verification**: You can browse the repository via `http://<EC2-IP>:8081` to see your versioned folders and files.

---------------------------------------------------------------------------
** Key Concepts:
**Repository Types**:
    *   **Hosted**: Your private storage for internal builds (what we used).
    *   **Proxy**: A local cache for public libraries (like NPM or Maven) to speed up builds.
    *   **Group**: A single URL that combines multiple repositories.
**Artifacts vs. Source Code**:
    *   *Source Code* (GitHub): Human-readable logic.
    *   *Artifact* (Nexus): Deployment-ready package (the "Final Product").

**  Deployment vs. Storage:
    *   **Nexus**: Stores the artifact securely and versioned.
    *   **Docker**: Runs the artifact as a live application on a server.
-----------------------------------------------------------------------------------
**Important Note:** 
*   **Nexus (Port 8081)** is for **Storing** the package. You cannot "run" the website from here.
*   **Docker (Port 8085)** is for **Running** the application. The live website is visible here.