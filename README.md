# ExternIT Reusable Workflows

Workflows GitHub Actions réutilisables pour tous les services ExternIT.

## Workflows disponibles

| Fichier | Description |
|---------|-------------|
| `docker-build.yml` | Build Docker + Trivy + push avec attestations |
| `php-tests.yml` | Tests PHPUnit (sans base de données) |
| `php-tests-db.yml` | Tests PHPUnit avec service de base de données |
| `js-tests.yml` | Tests JavaScript/TypeScript avec couverture |

---

## docker-build.yml

```yaml
jobs:
  docker:
    uses: ExternIT-repo/github-workflows/.github/workflows/docker-build.yml@main
    with:
      registry: docker.io                      # ou ghcr.io
      image-name: myorg/myapp
      push: ${{ github.ref == 'refs/heads/main' }}
      with-trivy: true
      smoke-test: |
        docker run --rm local-test:ci php -r 'echo "OK";'
    secrets:
      registry-username: ${{ secrets.DOCKERHUB_USERNAME }}
      registry-token: ${{ secrets.DOCKERHUB_TOKEN }}
```

**Inputs**

| Nom | Type | Défaut | Description |
|-----|------|--------|-------------|
| `registry` | string | — | Registre cible |
| `image-name` | string | — | Nom image sans registre |
| `dockerfile` | string | `Dockerfile` | Chemin du Dockerfile |
| `context` | string | `.` | Contexte de build |
| `push` | bool | `true` | Pousser après les tests |
| `with-trivy` | bool | `true` | Scan CVE Trivy |
| `trivy-ignore-file` | string | `.trivyignore` | Fichier d'exceptions Trivy |
| `smoke-test` | string | `` | Commande de smoke test (image = `local-test:ci`) |

---

## php-tests.yml

```yaml
jobs:
  tests:
    uses: ExternIT-repo/github-workflows/.github/workflows/php-tests.yml@main
    with:
      php-version: '8.2'
      test-command: 'composer test:coverage'
      coverage-type: badge        # badge | none
```

---

## php-tests-db.yml

```yaml
jobs:
  tests:
    uses: ExternIT-repo/github-workflows/.github/workflows/php-tests-db.yml@main
    with:
      php-version: '8.4'
      test-command: 'composer test-with-coverage'
      bootstrap-command: 'composer bootstrap-test-environment'
      db-image: 'mysql:8.4'
      db-name: 'myapp_test'
      database-url: 'mysql://root:test@127.0.0.1:3306/myapp_test?serverVersion=8.4'
      coverage-type: badge
```

---

## js-tests.yml

```yaml
jobs:
  tests:
    uses: ExternIT-repo/github-workflows/.github/workflows/js-tests.yml@main
    with:
      node-version: '22'
      test-command: 'npm run test:coverage'
      coverage-type: badge
```

---

## Triggers recommandés (éviter les doublons)

```yaml
on:
  pull_request:
    paths-ignore: ['badges/**']
  push:
    branches: [main, dev]
    paths-ignore: ['badges/**']
```

Les badges ne sont commités que sur `push` (jamais sur `pull_request`).
