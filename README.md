# Terratags GitHub Action

Validate tags on AWS, Azure, and Google Cloud resources in Terraform configurations using [Terratags](https://github.com/terratags/terratags).

## Quick Start

```yaml
- uses: terratags/terratags-action@v1
  with:
    config: '.terratags.yaml'
```

## Complete Workflow Example

```yaml
name: Validate Terraform Tags
on:
  pull_request:
    paths: ['**/*.tf']

jobs:
  validate-tags:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Validate Terraform tags
        uses: terratags/terratags-action@v1
        with:
          config: '.terratags.yaml'
          dir: './infrastructure'
          report: 'tag-report.html'
          remediate: true
          
      - name: Upload report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: tag-report
          path: tag-report.html
```

## Configuration

### Required Inputs

| Input | Description | Example |
|-------|-------------|---------|
| `config` | Path or URL to config file | `.terratags.yaml` |

### Optional Inputs

| Input | Description | Default | Example |
|-------|-------------|---------|---------|
| `dir` | Terraform directory to analyze | `.` | `./terraform` |
| `verbose` | Enable verbose output | `false` | `true` |
| `log-level` | Logging level | `ERROR` | `DEBUG` |
| `plan` | Terraform plan JSON file | - | `plan.json` |
| `report` | HTML report output path | - | `report.html` |
| `remediate` | Show remediation suggestions | `false` | `true` |
| `exemptions` | Exemptions file path | - | `exemptions.yaml` |
| `ignore-case` | Ignore case in tag keys | `false` | `true` |

### Outputs

| Output | Description |
|--------|-------------|
| `report-path` | Path to generated HTML report |

## Configuration File

Create `.terratags.yaml`:

```yaml
required_tags:
  - Environment
  - Team
  - CostCenter

patterns:
  Environment:
    - "^(dev|staging|prod)$"
  Team:
    - "^[a-z-]+$"

exemptions:
  - resource_type: "aws_s3_bucket"
    resource_name: "logs-bucket"
    reason: "Legacy bucket"
```

## Usage Examples

### Basic Validation
```yaml
- uses: terratags/terratags-action@v1
  with:
    config: '.terratags.yaml'
```

### With Terraform Plan Analysis
```yaml
- name: Generate Terraform plan
  run: terraform plan -out=plan.tfplan && terraform show -json plan.tfplan > plan.json

- uses: terratags/terratags-action@v1
  with:
    config: '.terratags.yaml'
    plan: 'plan.json'
    report: 'compliance-report.html'
```

### Remote Configuration
```yaml
- uses: terratags/terratags-action@v1
  with:
    config: 'https://github.com/org/configs.git//terratags.yaml?ref=main'
    verbose: true
    log-level: 'INFO'
```

### With Exemptions
```yaml
- uses: terratags/terratags-action@v1
  with:
    config: '.terratags.yaml'
    exemptions: 'exemptions.yaml'
    remediate: true
    ignore-case: true
```

## Configuration Examples

### AWS Resources
```yaml
required_tags:
  - Environment
  - Owner
  - Project

patterns:
  Environment:
    - "^(development|staging|production)$"
  Owner:
    - "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$"

providers:
  - aws
```

### Multi-Cloud
```yaml
required_tags:
  - Environment
  - Team

providers:
  - aws
  - azurerm
  - google

# GCP uses labels instead of tags
required_labels:
  - environment
  - team
```

## Troubleshooting

### Common Issues

**Config file not found:**
- Ensure the config file exists in the repository
- Check the file path is correct
- Verify file permissions

**No resources found:**
- Ensure Terraform files exist in the specified directory
- Check if resources are supported by Terratags
- Use verbose mode for detailed output

**Pattern validation fails:**
- Verify regex patterns are valid
- Test patterns with expected values
- Use ignore-case if needed

### Debug Mode
```yaml
- uses: terratags/terratags-action@v1
  with:
    config: '.terratags.yaml'
    verbose: true
    log-level: 'DEBUG'
```

## License

This action is distributed under the MIT License.
