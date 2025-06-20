# Paperless-NGX Suite Helm Chart

This Helm chart deploys a complete Paperless-NGX suite including both the main Paperless-NGX application and the AI-powered enhancement service (Paperless-AI).

## Features

- **Paperless-NGX**: Document management system with OCR capabilities
- **Paperless-AI**: AI-powered document analysis and enhancement
- **Configurable Backends**: Support for PostgreSQL/MariaDB and Redis
- **Flexible Storage**: Multiple persistent volume configurations
- **AI/ML Integration**: Support for multiple LLM providers (OpenAI, Anthropic, Ollama, Hugging Face)
- **Vector Database**: Support for Chroma, Pinecone, and Weaviate
- **Ingress Support**: Optional ingress configuration for both services

## Prerequisites

- Kubernetes 1.19+
- Helm 3.2.0+
- PV provisioner support in the underlying infrastructure

## Installing the Chart

### From your GitHub Pages Helm repository

First, add the published repo and update your local index:

```bash
helm repo add paperless https://michael-pisman.github.io/paperless-ngx-suite
helm repo update
```

To install the chart with the release name `paperless`:

```bash
helm install paperless ./paperless-ngx-suite
```

To install with custom values:

```bash
helm install paperless ./paperless-ngx-suite -f my-values.yaml
```

## Configuration

### Global Configuration

The following table lists the configurable global parameters:

| Parameter | Description | Default |
|-----------|-------------|---------|
| `global.database.type` | Database type (postgresql/mariadb) | `postgresql` |
| `global.database.host` | Database host | `""` |
| `global.database.port` | Database port | `5432` |
| `global.database.name` | Database name | `paperless` |
| `global.database.username` | Database username | `paperless` |
| `global.database.password` | Database password | `""` |
| `global.redis.enabled` | Enable Redis | `true` |
| `global.redis.host` | Redis host | `""` |
| `global.redis.port` | Redis port | `6379` |
| `global.redis.password` | Redis password | `""` |

### Paperless-NGX Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `paperless-ngx.replicaCount` | Number of replicas | `1` |
| `paperless-ngx.image.repository` | Image repository | `ghcr.io/paperless-ngx/paperless-ngx` |
| `paperless-ngx.image.tag` | Image tag | `latest` |
| `paperless-ngx.service.type` | Service type | `ClusterIP` |
| `paperless-ngx.service.port` | Service port | `8000` |
| `paperless-ngx.ingress.enabled` | Enable ingress | `false` |
| `paperless-ngx.persistence.enabled` | Enable persistence | `true` |
| `paperless-ngx.persistence.size` | Size of data volume | `20Gi` |
| `paperless-ngx.persistence.media.size` | Size of media volume | `50Gi` |
| `paperless-ngx.config.adminUser` | Admin username | `admin` |
| `paperless-ngx.config.adminMail` | Admin email | `admin@example.com` |
| `paperless-ngx.config.ocrLanguage` | OCR language | `eng` |
| `paperless-ngx.config.timeZone` | Timezone | `UTC` |

### Paperless-AI Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `paperless-ai.replicaCount` | Number of replicas | `1` |
| `paperless-ai.image.repository` | Image repository | `clusterzx/paperless-ai` |
| `paperless-ai.image.tag` | Image tag | `latest` |
| `paperless-ai.service.type` | Service type | `ClusterIP` |
| `paperless-ai.service.port` | Service port | `8080` |
| `paperless-ai.config.llm.provider` | LLM provider | `openai` |
| `paperless-ai.config.llm.model` | LLM model | `gpt-3.5-turbo` |
| `paperless-ai.config.llm.apiKey` | LLM API key | `""` |
| `paperless-ai.config.embeddings.provider` | Embeddings provider | `openai` |
| `paperless-ai.config.vectorDb.provider` | Vector DB provider | `chroma` |
| `paperless-ai.persistence.vectorDb.size` | Vector DB volume size | `20Gi` |

## Example Configurations

### Basic Setup with OpenAI

```yaml
global:
  database:
    host: "postgresql.example.com"
    password: "mydbpassword"
  redis:
    host: "redis.example.com"
    password: "myredispassword"

paperless-ngx:
  config:
    adminPassword: "myadminpassword"
    url: "https://paperless.example.com"
  ingress:
    enabled: true
    hosts:
      - host: paperless.example.com
        paths:
          - path: /
            pathType: Prefix

paperless-ai:
  config:
    llm:
      apiKey: "sk-your-openai-api-key"
      model: "gpt-4"
    embeddings:
      model: "text-embedding-ada-002"
  ingress:
    enabled: true
    hosts:
      - host: paperless-ai.example.com
        paths:
          - path: /
            pathType: Prefix
```

### Setup with Ollama (Local LLM)

```yaml
paperless-ai:
  config:
    llm:
      provider: "ollama"
      model: "llama2"
    ollama:
      host: "ollama.example.com"
      port: 11434
      model: "llama2:13b"
    embeddings:
      provider: "sentence-transformers"
      model: "all-MiniLM-L6-v2"
```

### Setup with Hugging Face

```yaml
paperless-ai:
  config:
    llm:
      provider: "huggingface"
      model: "google/flan-t5-large"
    huggingface:
      apiKey: "hf_your-huggingface-token"
    embeddings:
      provider: "huggingface"
      model: "sentence-transformers/all-MiniLM-L6-v2"
```

## Storage Configuration

The chart supports multiple storage configurations:

### Paperless-NGX Volumes
- **data**: Application data and configuration
- **media**: Document files and thumbnails
- **export**: Document exports
- **consume**: Documents to be processed

### Paperless-AI Volumes
- **data**: Application data
- **vectordb**: Vector database storage
- **models**: AI model cache

## Dependencies

This chart includes optional Bitnami charts for PostgreSQL and Redis:

### PostgreSQL (Bitnami)

```yaml
postgresql:
  enabled: true
  auth:
    postgresPassword: "mypassword"
    username: "paperless"
    password: "paperlesspassword"
    database: "paperless"
  primary:
    persistence:
      enabled: true
      size: 50Gi
      storageClass: "fast-ssd"
    resources:
      limits:
        cpu: 2000m
        memory: 4Gi
      requests:
        cpu: 1000m
        memory: 2Gi
  metrics:
    enabled: true
```

### Redis (Bitnami)

```yaml
redis:
  enabled: true
  auth:
    enabled: true
    password: "redispassword"
  master:
    persistence:
      enabled: true
      size: 5Gi
      storageClass: "fast-ssd"
    resources:
      limits:
        cpu: 1000m
        memory: 2Gi
      requests:
        cpu: 500m
        memory: 1Gi
  metrics:
    enabled: true
```

## Installation with Dependencies

To install with built-in PostgreSQL and Redis:

```bash
# Add Bitnami repository
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# Update dependencies
helm dependency update ./paperless-ngx-suite

# Install with dependencies enabled
helm install paperless ./paperless-ngx-suite \
  --set postgresql.enabled=true \
  --set redis.enabled=true \
  --set postgresql.auth.password="mydbpassword" \
  --set redis.auth.password="myredispassword"
```

## Upgrading

To upgrade the chart:

```bash
helm upgrade paperless ./paperless-ngx-suite
```

## Uninstalling

To uninstall the chart:

```bash
helm uninstall paperless
```

## Support

For issues and support, please refer to:
- [Paperless-NGX Documentation](https://docs.paperless-ngx.com/)
- [Paperless-NGX GitHub](https://github.com/paperless-ngx/paperless-ngx)

## License

This chart is licensed under the MIT License.