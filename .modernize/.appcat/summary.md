# Modernization Assessment Summary

**Target Azure Services**: Azure Kubernetes Service, Azure App Service, Azure Container Apps

## Overall Statistics

**Total Applications**: 1

**Name: movie-info**
- Mandatory: 9 issues
- Potential: 0 issues
- Optional: 1 issues

> **Severity Levels Explained:**
> - **Mandatory**: The issue has to be resolved for the migration to be successful.
> - **Potential**: This issue may be blocking in some situations but not in others. These issues should be reviewed to determine whether a change is required or not.
> - **Optional**: The issue discovered is real issue fixing which could improve the app after migration, however it is not blocking.

## Applications Profile

### Name: movie-info
- **JDK Version**: 1.8
- **Frameworks**: Spring Boot, Spring
- **Languages**: Java
- **Build Tools**: Maven

**Key Findings**:
- **Mandatory Issues (19 locations)**:
  - <!--ruleid=azure-java-version-02000-->Legacy Java version (1 location found)
  - <!--ruleid=spring-boot-to-azure-spring-boot-version-01000-->Spring Boot Version is End of OSS Support (4 locations found)
  - <!--ruleid=spring-framework-version-01000-->Spring Framework Version End of OSS Support (3 locations found)
  - <!--ruleid=localhost-http-00001-->Local HTTP Calls (2 locations found)
  - <!--ruleid=unsecure-network-protocol-00000-->Use of unsecured network protocols or URI libraries (3 locations found)
  - <!--ruleid=embedded-cache-16000-->Caching - Redis Cache library (1 location found)
  - <!--ruleid=java-11-deprecate-javaee-00001-->The java.annotation (Common Annotations) module has been removed from OpenJDK 11 (1 location found)
  - <!--ruleid=lombok-incompatibility-00001-->The Lombok version is incompatible with Open JDK 17 (1 location found)
  - <!--ruleid=java-19-deprecate-thread-00001-->This method is not final and may be overridden to return a value that is not the thread ID. Use Thread.threadId() instead. (3 locations found)
- **Optional Issues (4 locations)**:
  - <!--ruleid=hardcoded-urls-00001-->Avoid using hardcoded URLs (HTTP protocol) in source code (4 locations found)

## Next Steps

For comprehensive migration guidance and best practices, visit:
- [GitHub Copilot modernization](https://aka.ms/ghcp-appmod)
