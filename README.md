# github-actions-01-kubernetes

GitOps repository for the [rest-api-books](https://github.com/Orb1979/github-actions-test)
application: this repository holds the desired state of the local Kubernetes
cluster. **Nothing here is deployed by hand** — Argo CD watches `k8s/` and
syncs every change into the cluster.

## Layout

```
k8s/
├── app/
│   ├── deployment.yaml    application Deployment; image tag pinned by CI
│   └── service.yaml       LoadBalancer → http://localhost:8080 (Rancher Desktop)
└── postgres/
    ├── deployment.yaml     PostgreSQL 17 (books database)
    ├── service.yaml       cluster-internal 'postgres:5432' endpoint
    └── pvc.yaml           persistent volume for the database
```

## How this repository is updated

After a successful build, the `deploy` job in the application repository's
`publish-image.yml` workflow rewrites the image tag in `k8s/app/deployment.yaml`
to the freshly published `ghcr.io/orb1979/rest-api-books:sha-<commit>` and
pushes a commit. Argo CD picks up the commit within ~3 minutes and syncs the
cluster. The commit history of this repository is therefore the deployment log.

## Manual verification

```bash
kubectl apply -f k8s/
kubectl rollout status deployment/rest-api-books
curl http://localhost:8080/api/books
```
