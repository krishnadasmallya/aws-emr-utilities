---
name: "spark-upgrade-agent"
displayName: "Upgrade Spark applications on AWS"
description: "Accelerate Apache Spark version upgrades with conversational AI that automates code transformation, dependency resolution, and validation testing. Supports PySpark and Scala on EMR EC2 and EMR Serverless."
keywords: ["spark", "emr", "upgrade", "migration", "automation", "code-transformation", "pyspark", "scala", "aws"]
author: "AWS"
---

# Onboarding

## Step 1: Validate Prerequisites

Before proceeding, ensure the following are installed:

- **AWS CLI**: Required for AWS authentication
  - Verify with: `aws --version`
  - Install: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

- **Python 3.10+**: Required for MCP proxy
  - Verify with: `python3 --version`
  - Install: https://www.python.org/downloads/release/python-3100/

- **uv package manager**: Required for MCP Proxy for AWS
  - Verify with: `uv --version`
  - Install: https://docs.astral.sh/uv/getting-started/installation/

- **AWS Credentials**: Must be configured
  - Verify with: `aws sts get-caller-identity`
  - Configure via AWS CLI, environment variables, or IAM roles
  - Identify the AWS Profile that the creds are configured for
    - You can do this with `echo ${AWS_PROFILE}`. No response likely means they are using the `default` profile
    - use this profile later when configuring the `spark-upgrade-profile`

**CRITICAL**: If any prerequisites are missing, DO NOT proceed. Install missing components first.

## Step 2: Deploy CloudFormation Stack

**Note**: If the customer already has a role configured with `sagemaker-unified-studio-mcp` permissions, please ignore this step, get the AWS CLI profile for that role from the customer and configure your (Kiro's) mcp.json file (typically located at `~/.kiro/settings/mcp.json`) with that profile. However, it is recommended to deploy the stack as will contain latest updates for permissions and role policies

1. Log into the AWS Console with the role your AWS CLI is configured with - this role must have permissions to deploy Cloud Formation stacks.
1. Navigate to [upgrade agent setup page](https://docs.aws.amazon.com/emr/latest/ReleaseGuide/emr-spark-upgrade-agent-setup.html#spark-upgrade-agent-setup-resources).
1. Deploy Cfn stack / resources to desired region
1. **Configure parameters** on the "Specify stack details" page:
  - **CloudWatchKmsKeyArn**: (Optional) ARN of the KMS key used to encrypt EMR-Serverless CloudWatch Logs
  - **EMRServerlessS3LogPath**: (Optional) S3 path where EMR-Serverless application logs are stored
  - **EnableEMREC2**: Enable EMR-EC2 upgrade permissions
  - **EnableEMRServerless**: Enable EMR-Serverless upgrade permissions
  - **ExecutionRoleToGrantS3Access**: IAM Role Name or ARN of your EMR-EC2/EMR-Serverless job execution role that needs S3 staging bucket access
  - **S3KmsKeyArn**: (Optional) ARN of existing KMS key for S3 staging bucket encryption. Only used if UseS3Encryption is true and you have an existing bucket with a KMS key
  - **SparkUpgradeIAMRoleName**: Name of the IAM role to create for Spark upgrades. Leave empty to auto-generate with stack name for uniqueness
  - **StagingBucketPath**: S3 path for staging artifacts
  - **UseS3Encryption**: Enable KMS encryption for S3 staging bucket. Set to true to use KMS encryption instead of default S3 encryption
1. **Wait for deployment** to complete successfully.

## Step 3: Configure the AWS CLI with the MCP role and Kiro's mcp.json

1. Propmt the user to provide the region and role ARN from the Cfn stack deployment
1. Configure the CLI with the `spark-upgrade-profile`
    ```bash
    # run these as a single command so the use doesn't have to approve multiple times
    aws configure set profile.spark-upgrade-profile.source_profile <profile with Cloud Formation deployment permissions from earlier>
    aws configure set profile.spark-upgrade-profile.role_arn <role arn from Cloud Formation stack, provided by customer>
    aws configure set profile.spark-upgrade-profile.region <region from Cloud Formation stack, provided by customer>
    ```
1. then update your MCP json configuration file (typically located at `~/.kiro/settings/mcp.json`) with the region. Only edit your mcp.json, no other local copies, just the one that configures your mcp settings.

# Overview

The Apache Spark Upgrade Agent for Amazon EMR is a conversational AI capability that accelerates Apache Spark version upgrades for your EMR applications. Traditional Spark upgrades require months of engineering effort to analyze API changes, resolve dependency conflicts, and validate functional correctness. The agent simplifies the upgrade process through natural language prompts, automated code transformation, and data quality validation.

**Key capabilities:**
- **Automated Code Transformation**: Automatically updates code for Spark version compatibility
- **Dependency Resolution**: Resolves version conflicts and updates build configurations
- **Validation Testing**: Submits and monitors remote validation jobs on EMR
- **Data Quality Validation**: Ensures data integrity throughout the upgrade process
- **Natural Language Interface**: Interact using conversational prompts
- **Multi-Platform Support**: EMR EC2 and EMR Serverless
- **Multi-Language Support**: PySpark (Python) and Scala

**Note**: Preview service using cross-region inference for AI processing.

## Architecture Overview

The upgrade agent orchestrates the upgrade using specialized tools through these steps:

1. **Planning**: Analyzes project structure and generates upgrade plans
2. **Compile and Build**: Updates build environment, dependencies, and fixes build failures
3. **Spark Code Edit Tools**: Applies targeted code updates for version compatibility
4. **Execute & Validation**: Submits remote validation jobs to EMR and monitors execution
5. **Observability**: Tracks upgrade progress and provides status updates

## Available MCP Server

### spark-upgrade
**Connection:** MCP Proxy for AWS
**Authentication:** AWS IAM role assumption via spark-upgrade-profile
**Timeout:** 180 seconds

Provides comprehensive Spark upgrade tools including:
- Project analysis and upgrade planning
- Automated code transformation
- Build environment updates
- Dependency resolution
- Remote EMR job validation
- Data quality validation
- Progress tracking and observability

## Usage Examples

### Start a Spark Upgrade Project

```
"I want to upgrade my PySpark application from Spark 3.3 to 3.5. 
The project is located at /path/to/my/spark/project.
Can you help me create an upgrade plan?"
```

The agent will:
1. Analyze your project structure
2. Identify current Spark version and dependencies
3. Generate a comprehensive upgrade plan
4. Outline required changes and validation steps

### Automated Code Transformation

```
"Please apply the automated code transformations for my Spark 3.5 upgrade.
Fix any API compatibility issues and update deprecated methods."
```

The agent will:
1. Scan your codebase for compatibility issues
2. Apply automated code transformations
3. Update deprecated API calls
4. Fix build-time and runtime errors
5. Maintain approval control over all changes

### Build Environment Update

```
"Update my build configuration and dependencies for Spark 3.5.
My project uses Maven/SBT and has custom dependencies."
```

The agent will:
1. Update build files (pom.xml, build.sbt, etc.)
2. Resolve dependency conflicts
3. Update Spark and related library versions
4. Compile the project and fix build failures

### Remote Validation Testing

```
"Submit a validation job to EMR to test my upgraded application.
Use my existing EMR cluster j-XXXXX or create a new one with Spark 3.5."
```

The agent will:
1. Package your upgraded application
2. Submit validation jobs to EMR
3. Monitor job execution and logs
4. Report on success/failure and performance
5. Identify any runtime issues

### Data Quality Validation

```
"Validate that my upgraded Spark job produces the same data quality results.
Compare output between Spark 3.3 and 3.5 versions."
```

The agent will:
1. Run data quality checks on both versions
2. Compare output datasets
3. Identify any data discrepancies
4. Provide detailed validation reports

## Common Upgrade Scenarios

### PySpark Application Upgrade

```
"I have a PySpark ETL pipeline that processes daily data on EMR.
Current version: Spark 3.3, Target: Spark 3.5
Can you help me upgrade and validate it?"
```

Agent provides:
- Code compatibility analysis
- Automated PySpark API updates
- Dependency resolution
- EMR job validation
- Performance comparison

### Scala Spark Application Upgrade

```
"My Scala Spark streaming application needs to be upgraded from 3.2 to 3.5.
It uses structured streaming and custom serializers."
```

Agent handles:
- Scala API compatibility
- Structured streaming changes
- Serialization updates
- Build configuration updates
- Streaming job validation

### Multi-Module Project Upgrade

```
"I have a complex Spark project with multiple modules and shared libraries.
How should I approach upgrading this to Spark 3.5?"
```

Agent provides:
- Module dependency analysis
- Phased upgrade planning
- Shared library compatibility
- Integration testing strategy
- Coordinated validation approach

### EMR Serverless Migration with Upgrade

```
"I want to upgrade my Spark application and migrate from EMR on EC2 to EMR Serverless.
What's the best approach for this dual migration?"
```

Agent assists with:
- Platform-specific considerations
- Spark version compatibility
- Resource configuration changes
- Code modifications for serverless
- Validation on both platforms

## Best Practices

### ✅ Do:

- **Start with analysis** - Let the agent analyze your project first
- **Review all changes** - Approve each automated transformation
- **Test incrementally** - Validate changes step by step
- **Use staging environment** - Test on non-production EMR clusters first
- **Backup your code** - Commit changes to version control
- **Monitor validation jobs** - Watch EMR job execution closely
- **Validate data quality** - Ensure output correctness
- **Document the process** - Keep track of changes made
- **Plan for rollback** - Have a way to revert if needed
- **Engage early** - Start upgrade planning well in advance

### ❌ Don't:

- **Skip project analysis** - Always let the agent analyze first
- **Auto-approve all changes** - Review each transformation
- **Rush the process** - Take time to validate each step
- **Ignore build failures** - Fix compilation issues before proceeding
- **Skip validation testing** - Always test on EMR before production
- **Ignore data quality** - Validate output correctness
- **Forget dependencies** - Check all third-party libraries
- **Skip documentation** - Document your upgrade process
- **Ignore warnings** - Address compatibility warnings
- **Upgrade production directly** - Always test in staging first

## Troubleshooting

### "Project analysis failed"
**Cause:** Unable to access or parse project structure
**Solution:**
- Verify project path is correct and accessible
- Ensure project has valid build files (pom.xml, build.sbt, etc.)
- Check file permissions
- Provide more specific project information

### "Code transformation failed"
**Cause:** Complex code patterns that require manual intervention
**Solution:**
- Review the specific transformation error
- Apply manual fixes for complex cases
- Break down large transformations into smaller steps
- Consult Spark migration guides for specific patterns

### "Build compilation failed"
**Cause:** Dependency conflicts or incompatible versions
**Solution:**
- Review dependency version matrix
- Update conflicting libraries
- Check for Spark-specific dependency requirements
- Consider excluding transitive dependencies

### "EMR validation job failed"
**Cause:** Runtime errors in upgraded application
**Solution:**
- Review EMR job logs for specific errors
- Check Spark configuration compatibility
- Verify data input formats and schemas
- Test with smaller datasets first
- Apply additional code fixes based on runtime errors

### "Data quality validation failed"
**Cause:** Output differences between Spark versions
**Solution:**
- Review data quality report details
- Check for behavioral changes in Spark versions
- Verify input data consistency
- Adjust validation thresholds if appropriate
- Investigate specific data discrepancies

## Configuration

**Authentication:** AWS IAM role via CloudFormation stack (spark-upgrade-profile)

**Required Permissions:**
- EMR: `elasticmapreduce:*` (for job submission and monitoring)
- S3: `s3:GetObject`, `s3:PutObject` (for staging artifacts and logs)
- IAM: `iam:PassRole` (for EMR job execution)
- CloudWatch: `logs:*` (for log access and monitoring)

**Supported Platforms:**
- Amazon EMR on EC2
- Amazon EMR Serverless

**Supported Languages:**
- PySpark (Python)
- Scala

**Supported Build Systems:**
- Maven (pom.xml)
- SBT (build.sbt)
- Gradle (build.gradle)

## Tips

1. **Start with project analysis** - Always begin with "analyze my project"
2. **Review each transformation** - Don't auto-approve all changes
3. **Test incrementally** - Validate each step before proceeding
4. **Use natural language** - Describe your upgrade goals clearly
5. **Provide context** - Share project structure and requirements
6. **Monitor validation jobs** - Watch EMR execution closely
7. **Validate data quality** - Ensure output correctness
8. **Document changes** - Keep track of modifications
9. **Plan for rollback** - Have a revert strategy ready
10. **Engage early** - Start planning upgrades well in advance

---

**Service:** Amazon SageMaker Unified Studio MCP (Preview)  
**Provider:** AWS  
**License:** AWS Service Terms