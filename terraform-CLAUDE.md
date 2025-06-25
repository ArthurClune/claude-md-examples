# Terraform Development Instructions for Claude Code

## Overview
This guide defines how to work with Terraform code, emphasising multi-file coordination, linting with tflint, and security scanning with checkov.

## Critical Rules

### 1. Multi-File Awareness
- **Always scan the entire module directory** before making changes
- Check for dependencies across files: variables, outputs, data sources, and resources may be referenced anywhere
- When modifying a resource, search for all references to it across `.tf` files
- Consider module boundaries and inter-module dependencies

### 2. File Organisation
Follow standard Terraform conventions:
- `main.tf` - Primary resources
- `variables.tf` - Variable declarations
- `outputs.tf` - Output definitions
- `providers.tf` - Provider configurations
- `versions.tf` - Terraform and provider version constraints
- `data.tf` - Data sources (if numerous)
- `locals.tf` - Local values (if numerous)

### 3. Change Coordination
When making changes:
1. **Identify scope**: List all files that might be affected
2. **Search references**: Use grep/search for resource names, variable names, output names
3. **Update systematically**: Make changes in dependency order
4. **Verify completeness**: Re-check all references after changes

## Linting with tflint

### Setup Check
```bash
# Verify tflint is installed
which tflint || echo "tflint not installed"

# Check for .tflint.hcl configuration
ls -la .tflint.hcl
```

### Standard Linting Process
```bash
# Initialise tflint (first time or when plugins change)
tflint --init

# Run linting on current directory
tflint

# Run with more detailed output
tflint --format compact

# Run recursively on all modules
tflint --recursive
```

### Common tflint Issues to Fix
- Deprecated syntax or arguments
- Invalid instance types
- Hardcoded credentials
- Missing required providers
- Unused declarations

## Security Scanning with checkov

### Setup Check
```bash
# Verify checkov is installed
which checkov || echo "checkov not installed"

# Check Python version (needs 3.7+)
python3 --version
```

### Standard Security Scan
```bash
# Scan current directory
checkov -d .

# Scan with specific framework
checkov -d . --framework terraform

# Output results in JSON for parsing
checkov -d . -o json

# Skip specific checks
checkov -d . --skip-check CKV_AWS_20,CKV_AWS_23

# Run only specific checks
checkov -d . --check CKV_AWS_*
```

### Critical Security Checks
Focus on these high-priority items:
- Encryption at rest enabled
- Encryption in transit enabled
- Public access restrictions
- IAM least privilege
- Logging enabled
- Backup configured
- No hardcoded secrets

## Development Workflow

### 1. Before Starting Changes
```bash
# Check current state
terraform init
terraform validate
tflint
checkov -d .

# Review existing structure
find . -name "*.tf" -type f | head -20
grep -r "resource\|module\|data" --include="*.tf" | head -20
```

### 2. Making Changes
1. **Plan the change**: Identify all affected files
2. **Search for dependencies**:
   ```bash
   # Example: changing a security group
   grep -r "aws_security_group\.example" --include="*.tf"
   grep -r "security_group_id" --include="*.tf"
   ```
3. **Update systematically**: Start with variables, then resources, then outputs
4. **Maintain consistency**: Keep naming conventions and patterns

### 3. After Changes
```bash
# Validate syntax
terraform fmt -recursive
terraform validate

# Check for issues
tflint --recursive
checkov -d .

# Plan to verify
terraform plan -out=plan.out
```

## Common Patterns

### Resource Naming
```hcl
# Use consistent prefixes and suffixes
resource "aws_instance" "web_server" {
  # Not: resource "aws_instance" "my-instance"
}

# Include environment in names
resource "aws_s3_bucket" "data_${var.environment}" {
  bucket = "${var.project}-data-${var.environment}"
}
```

### Variable Usage
```hcl
# Always declare variables with:
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"

  validation {
    condition     = contains(["t3.micro", "t3.small", "t3.medium"], var.instance_type)
    error_message = "Instance type must be t3.micro, t3.small, or t3.medium."
  }
}
```

### Output References
```hcl
# When adding outputs, check if they're used in:
# - Parent modules
# - Remote state data sources
# - External scripts
output "vpc_id" {
  description = "ID of the VPC"
  value       = aws_vpc.main.id
}
```

## Multi-File Change Examples

### Example 1: Adding a New Variable
1. Add to `variables.tf`:
   ```hcl
   variable "enable_monitoring" {
     description = "Enable detailed monitoring"
     type        = bool
     default     = false
   }
   ```

2. Search for where to use it:
   ```bash
   grep -r "monitoring\|instance" --include="*.tf"
   ```

3. Update `main.tf`:
   ```hcl
   resource "aws_instance" "web" {
     # ...
     monitoring = var.enable_monitoring
   }
   ```

### Example 2: Refactoring Security Groups
1. Identify all references:
   ```bash
   grep -r "security_group" --include="*.tf"
   grep -r "ingress\|egress" --include="*.tf"
   ```

2. Create new security group in `security.tf`
3. Update all resources using the old security group
4. Add outputs if needed
5. Remove old security group only after confirming no references

## Testing Checklist

Before considering any Terraform change complete:

- [ ] `terraform fmt -recursive` passes
- [ ] `terraform validate` passes
- [ ] `tflint --recursive` shows no errors
- [ ] `checkov -d .` critical issues resolved
- [ ] All variable references resolved
- [ ] All output consumers checked
- [ ] Module interfaces maintained
- [ ] Documentation updated
- [ ] Change tested with `terraform plan`

## Module Development

When working with modules:
1. Check `modules/*/variables.tf` for interface changes
2. Verify `modules/*/outputs.tf` for breaking changes
3. Search parent directories for module calls
4. Update module version constraints if needed

## Common Pitfalls

1. **Partial Updates**: Changing a resource name but not its references
2. **Missing Dependencies**: Not adding `depends_on` when needed
3. **Security Group Loops**: Circular references between security groups
4. **Count/For_each Changes**: These cause resource recreation
5. **Provider Aliases**: Forgetting to pass providers to modules

## Emergency Procedures

If terraform plan shows unexpected destruction:
1. **STOP** - Don't apply
2. Check for:
   - Changed resource names
   - Modified count/for_each expressions
   - Moved resources without using `moved` blocks
3. Use `terraform state mv` if needed
4. Consider using `-target` for partial applies

## Integration with CI/CD

Include these checks in automated pipelines:
```bash
#!/bin/bash
set -e

terraform fmt -check -recursive
terraform init -backend=false
terraform validate
tflint --recursive
checkov -d . --quiet --compact
```

## Remember

1. **Read before writing**: Understand the existing structure
2. **Search before changing**: Find all dependencies
3. **Lint before committing**: Run tflint and checkov
4. **Plan before applying**: Always review terraform plan output
5. **Document complex logic**: Add comments for non-obvious p

# Live changes

Claude MUST NOT ever make changes to systems (terraform apply)
***LINT and CHECK ONLY***
