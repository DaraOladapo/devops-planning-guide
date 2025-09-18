# CI/CD Implementation - Medium Business

## Overview

Medium business CI/CD requires enterprise-grade practices with standardization across multiple teams while maintaining flexibility. This section focuses on building scalable, secure, and compliant CI/CD pipelines that support multiple teams and applications.

## Enterprise CI/CD Architecture

### Multi-Team Pipeline Strategy

```
┌─────────────────────────────────────────────────────────────┐
│                 Shared CI/CD Platform                       │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐          │
│  │   GitLab    │ │   Jenkins   │ │    GitHub   │          │
│  │   Ultimate  │ │ Enterprise  │ │  Enterprise │          │
│  └─────────────┘ └─────────────┘ └─────────────┘          │
└─────────────────────┬───────────────────────────────────────┘
                      │
┌─────────────────────┴───────────────────────────────────────┐
│              Pipeline Orchestration                         │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐          │
│  │   Security  │ │ Compliance  │ │   Quality   │          │
│  │   Scanning  │ │   Gates     │ │   Gates     │          │
│  └─────────────┘ └─────────────┘ └─────────────┘          │
└─────────────────────┬───────────────────────────────────────┘
                      │
┌─────────────────────┴───────────────────────────────────────┐
│              Multi-Environment Deployment                   │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐          │
│  │    Dev      │ │   Staging   │ │ Production  │          │
│  │ Environment │ │ Environment │ │ Environment │          │
│  └─────────────┘ └─────────────┘ └─────────────┘          │
└─────────────────────────────────────────────────────────────┘
```

## Implementation Strategy

### Phase 1: Platform Standardization (Months 1-2)

#### CI/CD Platform Selection and Setup

**Recommended Enterprise Platforms:**

1. **GitLab Ultimate** (Recommended for integrated approach)
   - Complete DevOps platform
   - Built-in security scanning
   - Compliance features
   - Multi-project management

2. **Jenkins Enterprise** (For complex enterprise needs)
   - Maximum flexibility and plugins
   - Mature ecosystem
   - Strong enterprise support
   - Custom pipeline capabilities

3. **Azure DevOps Server** (For Microsoft-centric environments)
   - Integrated with Microsoft ecosystem
   - Strong project management
   - Hybrid cloud capabilities
   - Enterprise licensing

#### Infrastructure as Code for CI/CD

**Terraform Configuration Example:**
```hcl
# CI/CD Infrastructure
module "cicd_platform" {
  source = "./modules/cicd"
  
  environment = var.environment
  
  # Jenkins/GitLab cluster configuration
  cluster_config = {
    instance_type     = "c5.xlarge"
    min_size         = 3
    max_size         = 10
    availability_zones = ["us-east-1a", "us-east-1b", "us-east-1c"]
  }
  
  # Database for CI/CD platform
  database_config = {
    engine         = "postgresql"
    instance_class = "db.r5.large"
    storage        = 100
    backup_window  = "03:00-04:00"
  }
  
  # Load balancer configuration
  load_balancer = {
    ssl_certificate_arn = var.ssl_cert_arn
    health_check_path   = "/health"
  }
}

# Shared services
module "shared_services" {
  source = "./modules/shared"
  
  # Artifact repository
  artifact_repository = {
    type = "nexus"
    storage_size = "500GB"
  }
  
  # Container registry
  container_registry = {
    type = "harbor"
    replication_enabled = true
  }
  
  # Secret management
  secrets_manager = {
    type = "vault"
    ha_enabled = true
  }
}
```

### Phase 2: Pipeline Templates and Standards (Months 2-3)

#### Standardized Pipeline Templates

**Multi-Language Pipeline Template:**
```yaml
# .gitlab-ci.yml template for multiple technologies
stages:
  - validate
  - build
  - test
  - security
  - package
  - deploy-staging
  - integration-test
  - deploy-production

variables:
  DOCKER_REGISTRY: "$CI_REGISTRY"
  SONAR_HOST: "https://sonar.company.com"
  
# Include templates based on technology
include:
  - template: Security/SAST.gitlab-ci.yml
  - template: Security/Dependency-Scanning.gitlab-ci.yml
  - template: Security/Container-Scanning.gitlab-ci.yml
  - local: '/ci-templates/.java-template.yml'
    rules:
      - if: '$CI_PROJECT_PATH =~ /.*java.*/'
  - local: '/ci-templates/.nodejs-template.yml'
    rules:
      - if: '$CI_PROJECT_PATH =~ /.*node.*/'

# Security and compliance gates
security_scan:
  stage: security
  script:
    - sonar-scanner
    - dependency-check
    - container-scan
  artifacts:
    reports:
      sast: gl-sast-report.json
      dependency_scanning: gl-dependency-scanning-report.json
  only:
    - master
    - develop

# Deployment with approval gates
deploy_production:
  stage: deploy-production
  environment:
    name: production
    url: https://app.company.com
  when: manual
  only:
    - master
  before_script:
    - vault auth -method=aws
    - export DB_PASSWORD=$(vault kv get -field=password secret/db)
  script:
    - helm upgrade --install app ./helm-chart
    - run-smoke-tests.sh
```

#### Java/Spring Boot Template
```yaml
# .java-template.yml
.java_build:
  image: openjdk:11-jdk
  before_script:
    - ./mvnw dependency:go-offline
  cache:
    paths:
      - .m2/repository/
      - target/

java_test:
  extends: .java_build
  stage: test
  script:
    - ./mvnw test
    - ./mvnw jacoco:report
  artifacts:
    reports:
      junit: target/surefire-reports/TEST-*.xml
      coverage: target/site/jacoco/jacoco.xml
  coverage: '/Total.*?([0-9]{1,3})%/'

java_build:
  extends: .java_build
  stage: build
  script:
    - ./mvnw clean package -DskipTests
  artifacts:
    paths:
      - target/*.jar
    expire_in: 1 hour
```

#### Node.js Template
```yaml
# .nodejs-template.yml
.nodejs_base:
  image: node:16-alpine
  before_script:
    - npm ci --cache .npm --prefer-offline
  cache:
    paths:
      - .npm/
      - node_modules/

nodejs_test:
  extends: .nodejs_base
  stage: test
  script:
    - npm run lint
    - npm run test:coverage
  artifacts:
    reports:
      junit: coverage/junit.xml
      coverage: coverage/cobertura-coverage.xml
  coverage: '/All files[^|]*\|[^|]*\s+([\d\.]+)/'

nodejs_build:
  extends: .nodejs_base
  stage: build
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 hour
```

### Phase 3: Security and Compliance Integration (Months 3-4)

#### Automated Security Scanning

**Security Pipeline Integration:**
```yaml
# Security scanning stages
security_gates:
  stage: security
  parallel:
    matrix:
      - SCAN_TYPE: [sast, dependency, container, license]
  script:
    - case $SCAN_TYPE in
        sast) run_static_analysis.sh ;;
        dependency) check_dependencies.sh ;;
        container) scan_container_image.sh ;;
        license) check_licenses.sh ;;
      esac
  artifacts:
    reports:
      security: security-report-${SCAN_TYPE}.json
```

**Security Policy as Code:**
```yaml
# security-policies.yml
security_policies:
  sast:
    fail_on: [high, critical]
    allow_failure: false
    
  dependency_scanning:
    fail_on: [critical]
    allow_failure: false
    whitelist:
      - CVE-2021-12345  # Business approved exception
      
  container_scanning:
    fail_on: [high, critical]
    base_image_policy: company-approved-only
    
  license_scanning:
    allowed_licenses:
      - MIT
      - Apache-2.0
      - BSD-3-Clause
    forbidden_licenses:
      - GPL-3.0
      - AGPL
```

#### Compliance Automation

**SOX Compliance Pipeline:**
```yaml
sox_compliance:
  stage: compliance
  script:
    - validate_segregation_of_duties.sh
    - check_approval_gates.sh
    - verify_audit_logging.sh
    - validate_change_controls.sh
  artifacts:
    reports:
      compliance: sox-compliance-report.json
  only:
    variables:
      - $SOX_REGULATED == "true"
```

### Phase 4: Multi-Environment Strategy (Months 4-5)

#### Environment Management

**Environment Configuration:**
```yaml
# Environment-specific configurations
.deploy_template: &deploy_template
  image: kubectl:latest
  before_script:
    - kubectl config use-context $KUBE_CONTEXT
    - helm repo add company-charts https://charts.company.com
  script:
    - helm upgrade --install $APP_NAME company-charts/app
        --set image.tag=$CI_COMMIT_SHA
        --set environment=$ENVIRONMENT
        --values environments/$ENVIRONMENT/values.yaml
  after_script:
    - run_health_checks.sh
    - notify_deployment_status.sh

deploy_development:
  <<: *deploy_template
  stage: deploy-dev
  environment:
    name: development
    url: https://dev-$CI_PROJECT_NAME.company.com
  variables:
    ENVIRONMENT: "development"
    KUBE_CONTEXT: "dev-cluster"
  only:
    - develop

deploy_staging:
  <<: *deploy_template
  stage: deploy-staging
  environment:
    name: staging
    url: https://staging-$CI_PROJECT_NAME.company.com
  variables:
    ENVIRONMENT: "staging"
    KUBE_CONTEXT: "staging-cluster"
  only:
    - master

deploy_production:
  <<: *deploy_template
  stage: deploy-production
  environment:
    name: production
    url: https://app.company.com
  variables:
    ENVIRONMENT: "production"
    KUBE_CONTEXT: "prod-cluster"
  when: manual
  only:
    - master
```

## Advanced Deployment Strategies

### Blue-Green Deployments

**Blue-Green with Kubernetes:**
```yaml
# Blue-Green deployment script
blue_green_deploy:
  stage: deploy-production
  script:
    - |
      # Determine current and target colors
      CURRENT_COLOR=$(kubectl get service app-service -o jsonpath='{.spec.selector.version}')
      TARGET_COLOR=$([ "$CURRENT_COLOR" = "blue" ] && echo "green" || echo "blue")
      
      # Deploy to target color
      helm upgrade --install app-$TARGET_COLOR ./helm-chart 
        --set image.tag=$CI_COMMIT_SHA 
        --set version=$TARGET_COLOR
        --set service.enabled=false
      
      # Wait for deployment to be ready
      kubectl rollout status deployment/app-$TARGET_COLOR
      
      # Run smoke tests against target
      run_smoke_tests.sh "app-$TARGET_COLOR"
      
      # Switch traffic to target
      kubectl patch service app-service -p '{"spec":{"selector":{"version":"'$TARGET_COLOR'"}}}'
      
      # Wait and validate
      sleep 30
      run_production_tests.sh
      
      # Cleanup old deployment
      helm uninstall app-$CURRENT_COLOR
```

### Canary Deployments with Istio

**Canary Deployment Configuration:**
```yaml
# Istio canary deployment
canary_deploy:
  stage: deploy-production
  script:
    - |
      # Deploy canary version
      helm upgrade --install app-canary ./helm-chart 
        --set image.tag=$CI_COMMIT_SHA 
        --set canary.enabled=true
        --set canary.weight=10
      
      # Update Istio traffic rules
      kubectl apply -f - <<EOF
      apiVersion: networking.istio.io/v1alpha3
      kind: VirtualService
      metadata:
        name: app-traffic
      spec:
        http:
        - match:
          - headers:
              canary:
                exact: "true"
          route:
          - destination:
              host: app-canary
        - route:
          - destination:
              host: app-stable
            weight: 90
          - destination:
              host: app-canary
            weight: 10
      EOF
      
      # Monitor canary metrics
      monitor_canary_metrics.sh 10  # 10% traffic for 10 minutes
      
      # Promote or rollback based on metrics
      if [ $? -eq 0 ]; then
        promote_canary.sh
      else
        rollback_canary.sh
      fi
```

## Enterprise Tool Integration

### Monitoring and Observability

**Pipeline Monitoring Integration:**
```yaml
# Monitoring integration in pipelines
monitoring_setup:
  stage: deploy
  script:
    - |
      # Create Datadog deployment event
      curl -X POST "https://api.datadoghq.com/api/v1/events" \
        -H "Content-Type: application/json" \
        -H "DD-API-KEY: $DATADOG_API_KEY" \
        -d '{
          "title": "Deployment Started",
          "text": "Application '$CI_PROJECT_NAME' deployment to '$ENVIRONMENT' started",
          "tags": ["environment:'$ENVIRONMENT'", "project:'$CI_PROJECT_NAME'"]
        }'
      
      # Update deployment tracking
      update_deployment_dashboard.sh
      
      # Set up monitoring alerts
      configure_monitoring_alerts.sh $ENVIRONMENT
```

### Integration Testing Framework

**Comprehensive Integration Tests:**
```yaml
integration_tests:
  stage: integration-test
  services:
    - postgres:13
    - redis:6
    - elasticsearch:7.15.0
  variables:
    DATABASE_URL: "postgres://postgres:postgres@postgres:5432/testdb"
    REDIS_URL: "redis://redis:6379"
    ELASTICSEARCH_URL: "http://elasticsearch:9200"
  script:
    - setup_test_data.sh
    - run_api_tests.sh
    - run_performance_tests.sh
    - run_security_tests.sh
    - run_contract_tests.sh
  artifacts:
    reports:
      junit: test-results/integration-*.xml
      performance: performance-results.json
    paths:
      - test-results/
    expire_in: 7 days
```

## Performance and Optimization

### Pipeline Performance Optimization

**Caching Strategy:**
```yaml
# Advanced caching configuration
variables:
  CACHE_VERSION: "v1"

cache:
  key: "$CI_COMMIT_REF_SLUG-$CACHE_VERSION"
  paths:
    - .m2/repository/
    - node_modules/
    - ~/.cache/
  policy: pull-push

# Selective cache usage
build_job:
  cache:
    key: "$CI_COMMIT_REF_SLUG-build-$CACHE_VERSION"
    paths:
      - target/
    policy: push

test_job:
  cache:
    key: "$CI_COMMIT_REF_SLUG-build-$CACHE_VERSION"
    paths:
      - target/
    policy: pull
```

**Parallel Execution:**
```yaml
# Parallel test execution
test:
  parallel: 4
  script:
    - run_tests.sh --partition $CI_NODE_INDEX --total $CI_NODE_TOTAL
```

### Resource Management

**Resource Optimization:**
```yaml
# Resource-aware job configuration
.resource_small: &resource_small
  tags:
    - docker
    - small
  variables:
    KUBERNETES_CPU_REQUEST: "500m"
    KUBERNETES_MEMORY_REQUEST: "1Gi"

.resource_large: &resource_large
  tags:
    - docker
    - large
  variables:
    KUBERNETES_CPU_REQUEST: "2"
    KUBERNETES_MEMORY_REQUEST: "4Gi"

unit_tests:
  <<: *resource_small
  stage: test
  script: run_unit_tests.sh

integration_tests:
  <<: *resource_large
  stage: test
  script: run_integration_tests.sh
```

## Metrics and Analytics

### CI/CD Metrics Dashboard

**Key Metrics to Track:**
- **Pipeline Success Rate**: >95% target
- **Average Pipeline Duration**: <30 minutes target
- **Deployment Frequency**: Daily deployments per team
- **Lead Time**: <24 hours for feature delivery
- **Mean Time to Recovery**: <2 hours for rollbacks

**Metrics Collection:**
```yaml
# Metrics collection in pipeline
collect_metrics:
  stage: .post
  script:
    - |
      # Send metrics to monitoring system
      curl -X POST "$METRICS_ENDPOINT" \
        -H "Content-Type: application/json" \
        -d '{
          "pipeline_duration": "'$CI_PIPELINE_DURATION'",
          "pipeline_status": "'$CI_PIPELINE_STATUS'",
          "project": "'$CI_PROJECT_NAME'",
          "branch": "'$CI_COMMIT_REF_NAME'",
          "commit": "'$CI_COMMIT_SHA'"
        }'
  when: always
```

## Cost Management

### Cloud Cost Optimization

**Cost-Aware Pipeline Scheduling:**
```yaml
# Cost optimization strategies
cost_optimized_schedule:
  rules:
    - if: '$CI_PIPELINE_SOURCE == "schedule"'
      when: never  # Avoid unnecessary scheduled runs
    - if: '$CI_COMMIT_BRANCH == "master"'
      when: always
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
      changes:
        - "src/**/*"
        - "tests/**/*"
```

**Resource Cleanup:**
```yaml
cleanup_resources:
  stage: .post
  script:
    - cleanup_test_environments.sh
    - remove_temporary_resources.sh
    - optimize_artifact_storage.sh
  when: always
```

## Enterprise Integration

### Identity and Access Management

**LDAP/SSO Integration:**
```yaml
# Example GitLab LDAP configuration
production:
  ldap:
    enabled: true
    servers:
      main:
        label: 'Company LDAP'
        host: 'ldap.company.com'
        port: 636
        uid: 'sAMAccountName'
        encryption: 'simple_tls'
        base: 'DC=company,DC=com'
        user_filter: '(memberOf=CN=developers,OU=groups,DC=company,DC=com)'
```

### Enterprise Notifications

**Multi-Channel Notifications:**
```yaml
notifications:
  stage: .post
  script:
    - |
      # Slack notification
      send_slack_notification.sh "$CI_PIPELINE_STATUS" "$CI_PROJECT_NAME"
      
      # Email notification for failures
      if [ "$CI_PIPELINE_STATUS" = "failed" ]; then
        send_email_notification.sh
      fi
      
      # JIRA integration for production deployments
      if [ "$ENVIRONMENT" = "production" ]; then
        update_jira_tickets.sh
      fi
  when: always
```

## Next Steps

With enterprise CI/CD implemented, continue to:
[Test Management →](../05-Test-Management/)

---
*[← Previous: Process](../03-Process/) | [Back to Medium Business Guide](../README.md)*