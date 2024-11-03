# Complete CI/CD pipeline with EKS and DockerHub registry

# Technologies used:
Kubernetes, Jenkins, AWS EKS, Docker Hub, Java, Maven, Linux, Docker,Git

# Project Description:
Write K8s manifest files for Deployment and Service configuration
Integrate deploy step in the CI/CD pipeline to deploy newly built application image from
DockerHub private registry to the EKS cluster
So the complete CI/CD project we build has the following configuration:
a.CI step:Increment version
b.CI step: Build artifact for Java Maven application
c.CI step: Build and push Docker image to DockerHub
d.CD step: Deploy new application version to EKS cluster
e.CD step: Commit the version update

