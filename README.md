# LAMMPS Project Folder Structure

This project contains several components for building, running, and automating LAMMPS simulations with AWS SageMaker. Below is an overview of the folder structure and their purposes:


- **Sagemaker_dockerfile/**: Main Dockerfiles for SageMaker.  
  - To build the main Docker image, run from the root folder:  
    `docker build --platform linux/amd64 -t sm-build -f Sagemaker_dockerfile/GPU/Dockerfile .`  
    Replace `sm-build` with any image name of your choice.
- **Lightweight_dockerfile/**: Contains lightweight Dockerfiles for LAMMPS images that do *not* use the SageMaker distribution image as a base.
- **Lammps_project.ipynb**: Custom notebook/script to be run on a SageMaker JupyterLab instance. Automates simulation file execution, ensures files are tar/zipped, and uploads results to an S3 bucket.
- **LCC-auto-stop-idle/**: Contains scripts for SageMaker lifecycle configuration to auto-stop idle instances.  
  - For usage and setup, refer to:  
    https://github.com/aws-samples/sagemaker-studio-apps-lifecycle-config-examples/tree/main/jupyterlab/auto-stop-idle
- **Lammps_bundle_cpu/** and **Lammps_bundle_gpu/**:  
  *These folders are not included in this repository.*  
  You can download pre-built LAMMPS binaries for CPU and GPU from the [official LAMMPS website](https://lammps.org/download.html) or build them yourself. Place the downloaded or built binaries in these folders if needed for your workflow.


# Building and Deploying the LAMMPS SageMaker Custom Image

This guide explains how to build, tag, and push your custom LAMMPS Docker image for use with AWS SageMaker JupyterLab, using macOS and zsh shell. It also covers verifying the image with `skopeo`.

## Prerequisites
- Docker installed and running
- AWS CLI configured
- Skopeo installed (`brew install skopeo`)
- Access to an AWS ECR repository (replace placeholders with your values)

## 1. Build the Docker Image
```zsh
docker build linux/amd64 -t lammps-sagemaker:latest .
```

## 2. Tag the Image for ECR
```zsh
# Replace <aws_account_id> and <region> with your values
docker tag lammps-sagemaker:latest <aws_account_id>.dkr.ecr.<region>.amazonaws.com/lammps-sagemaker:latest
```

## 3. Authenticate Docker to ECR
```zsh
aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.<region>.amazonaws.com
```

## 4. Push the Image to ECR
```zsh
docker push <aws_account_id>.dkr.ecr.<region>.amazonaws.com/lammps-sagemaker:latest
```

## 5. Verify the Image with Skopeo
```zsh
skopeo inspect docker://<aws_account_id>.dkr.ecr.<region>.amazonaws.com/lammps-sagemaker:latest
```

## 6. Use the Image in SageMaker
- Register the ECR image URI in SageMaker Studio as a custom image.
- Launch a new JupyterLab instance using this image.

---

## Troubleshooting: Image Manifest Media Type for SageMaker Compatibility

AWS SageMaker requires Docker images to use the manifest type:

- `application/vnd.docker.distribution.manifest.v2+json` (Docker Image Manifest V2, Schema 2)

If your image is created or copied with a manifest type like V1 (`application/vnd.docker.distribution.manifest.v1+json`), SageMaker will not accept it.

### Why does this happen?
- Some tools (or older Docker versions) may create or copy images using the older V1 manifest.
- Skopeo, if not used with the correct options, may default to V1 when copying between registries or from local to remote.

### How to ensure V2 manifest (Schema 2) is used

#### 1. Always build and push with modern Docker (20.x or newer)

#### 2. If using `skopeo copy`, force the correct manifest type:

```zsh
# Example values:
AWS_ACCOUNT_ID=123456789012
REGION=us-east-1
REPO=lammps-sagemaker
TAG=latest

# Copy from local Docker to ECR with V2 manifest
skopeo copy --format v2s2 \
  docker-daemon:${REPO}:${TAG} \
  docker://${AWS_ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/${REPO}:${TAG}
```

#### 3. Verify manifest type after push

```zsh
aws ecr describe-images \
  --repository-name lammps-sagemaker \
  --image-ids imageTag=latest \
  --region us-east-1

```

You should see:
```
"imageManifestMediaType": "application/vnd.docker.distribution.manifest.v2+json",
```

#### 4. If you see V1 manifest, re-copy with the `--format v2s2` option as above.

---

## Example: Inspecting ECR Image Manifest with Skopeo (macOS/zsh)

To verify the manifest type of your image in ECR, use the following command (replace values as needed):

```zsh
skopeo inspect \
  --creds AWS:$(aws ecr get-login-password --region us-east-1) \
  docker://938398057556.dkr.ecr.us-east-1.amazonaws.com/lammps-sagemaker:latest
```

- This command authenticates to ECR and inspects the image manifest.
- Look for `"imageManifestMediaType": "application/vnd.docker.distribution.manifest.v2+json"` in the output.

**Tip:** You can use this command any time to check the manifest type of your ECR images for SageMaker compatibility.

---

**Summary:**
- Always use Docker or Skopeo with the correct options to ensure V2 manifest.
- SageMaker will only accept images with `application/vnd.docker.distribution.manifest.v2+json`.
- Use the above commands, replacing variables as needed for your account and region.

---

**Note:** Replace `<aws_account_id>` and `<region>` with your AWS account ID and region (e.g., `123456789012` and `us-east-1`).
