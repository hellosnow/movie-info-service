---
name: assessment
description: Analyze the application and generate assessment report
---

# Application Assessment Skill

This skill analyzes a Java/Spring Boot application and generates a comprehensive assessment report.

## Purpose

This skill performs a detailed assessment of the application to identify:
- Technology stack and versions
- Dependencies and frameworks
- Potential migration issues
- Cloud readiness concerns
- Upgrade opportunities
- Security vulnerabilities

## Inputs

- `workspace-path` (Mandatory): The root path of the project to assess
- `output-path` (Optional): The directory where the assessment report will be saved. Defaults to `.github/modernize/`
- `language` (Optional): The programming language of the project. Defaults to auto-detection.

## Workflow

1. **Analyze Project Structure**: 
   - Detect the programming language and build system
   - Identify the project type (Maven, Gradle, etc.)
   - Scan project files (pom.xml, build.gradle, package.json, etc.)

2. **Technology Stack Analysis**:
   - Identify frameworks and libraries
   - Determine versions of key dependencies
   - Check for deprecated or outdated dependencies

3. **Run Assessment Tools**:
   - For Java projects: Use AppCAT (Application Containerization and Migration Tool) to analyze the codebase
   - Scan for migration issues
   - Identify cloud readiness concerns
   - Check for security vulnerabilities

4. **Generate Report**:
   - Create a comprehensive assessment report in Markdown format
   - Include findings organized by category:
     - Technology Profile
     - Migration Issues
     - Cloud Readiness
     - Security Concerns
     - Recommendations
   - Save the report to the specified output path

5. **Summary**:
   - Provide a high-level summary of findings
   - Highlight critical issues that need immediate attention
   - Suggest next steps for modernization

## Output

The skill generates an `assessment-report.md` file in the specified output directory with:
- Executive summary of findings
- Detailed technology profile
- List of migration issues categorized by severity
- Cloud readiness assessment
- Security vulnerabilities (if any)
- Recommendations for modernization

## Completion Criteria

1. The assessment report is generated and saved to the output path
2. The report contains accurate information about the application
3. All critical issues are highlighted
4. Recommendations are actionable and prioritized
