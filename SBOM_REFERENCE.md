# SBOM Reference - Component Version Updates

This document provides information about the Software Bill of Materials (SBOM) files generated for all components in this monitoring stack.

## Overview

SBOM (Software Bill of Materials) files provide a comprehensive inventory of all software components, libraries, and dependencies in each Docker image. These files are essential for:
- Security vulnerability tracking
- License compliance
- Supply chain transparency
- Dependency management
- Audit and compliance requirements

All SBOM files are generated in **SPDX JSON format** using [Syft](https://github.com/anchore/syft) v1.36.0.

---

## SBOM Files

The following SBOM files are available in the `sbom/` directory:

### Component SBOMs

| Component | Version | SBOM File | Package Count | Generated |
|-----------|---------|-----------|---------------|-----------|
| Prometheus | v3.7.2 | `prometheus-v3.7.2-sbom.spdx.json` | 423 packages | 2025-10-30 |
| Grafana | 12.2.1 | `grafana-12.2.1-sbom.spdx.json` | 582 packages | 2025-10-30 |
| Alertmanager | v0.28.1 | `alertmanager-v0.28.1-sbom.spdx.json` | 157 packages | 2025-10-30 |
| Loki | 3.5.7 | `loki-3.5.7-sbom.spdx.json` | 302 packages | 2025-10-30 |
| Promtail | 3.5.7 | `promtail-3.5.7-sbom.spdx.json` | 349 packages | 2025-10-30 |
| Pushgateway | v1.11.1 | `pushgateway-v1.11.1-sbom.spdx.json` | 34 packages | 2025-10-30 |
| syslog-ng | 4.8.0 | `syslog-ng-4.8.0-sbom.spdx.json` | 287 packages | 2025-10-30 |
| PVE Exporter | 3.5.5 | `pve-exporter-3.5.5-sbom.spdx.json` | 90 packages | 2025-10-30 |
| jinja2docker | 2.1.8 | `jinja2docker-2.1.8-sbom.spdx.json` | 51 packages | 2025-10-30 |

**Total Packages Across All Components:** 2,275 packages

---

## File Format

All SBOM files follow the **SPDX 2.3** specification and are provided in JSON format for easy machine parsing and integration with security scanning tools.

### SPDX Format Benefits

- **Industry Standard**: SPDX is an ISO/IEC standard (ISO/IEC 5962:2021)
- **Tool Compatibility**: Supported by most vulnerability scanning tools
- **Comprehensive**: Includes package names, versions, licenses, and relationships
- **Machine-Readable**: JSON format for automation and CI/CD integration

---

## Using SBOM Files

### Vulnerability Scanning

You can use these SBOM files with vulnerability scanning tools:

#### With Grype (by Anchore)
```bash
grype sbom:sbom/prometheus-v3.7.2-sbom.spdx.json
```

#### With Trivy
```bash
trivy sbom sbom/grafana-12.2.1-sbom.spdx.json
```

#### With Docker Scout
```bash
docker scout cves --sbom sbom/loki-3.5.7-sbom.spdx.json
```

### License Compliance

Extract license information from SBOMs:

```bash
cat sbom/prometheus-v3.7.2-sbom.spdx.json | jq '.packages[] | {name: .name, license: .licenseDeclared}'
```

### Dependency Analysis

List all dependencies for a component:

```bash
cat sbom/grafana-12.2.1-sbom.spdx.json | jq '.packages[] | .name' | sort
```

---

## Component Package Breakdown

### Largest Components (by package count)

1. **Grafana** (582 packages)
   - Complex web application with numerous frontend and backend dependencies
   - Includes Node.js packages, Go modules, and system libraries

2. **Prometheus** (423 packages)
   - Time-series database with extensive Go ecosystem dependencies
   - Includes web UI components and various exporters

3. **Promtail** (349 packages)
   - Log collection agent with compression and forwarding capabilities
   - Shares dependencies with Loki

4. **Loki** (302 packages)
   - Log aggregation system
   - Similar dependency profile to Promtail

5. **syslog-ng** (287 packages)
   - Comprehensive logging framework
   - Includes various C/C++ system libraries

### Smallest Components (by package count)

1. **Pushgateway** (34 packages)
   - Focused, single-purpose component
   - Minimal dependencies

2. **jinja2docker** (51 packages)
   - Python-based templating tool
   - Limited scope and dependencies

3. **PVE Exporter** (90 packages)
   - Specialized exporter for Proxmox
   - Python-based with focused dependencies

---

## Security Scanning Recommendations

### Regular Scanning Schedule

- **Daily**: Scan production images for new vulnerabilities
- **Weekly**: Review SBOM changes and dependency updates
- **Monthly**: Comprehensive security audit of all components
- **On Update**: Scan immediately when updating any component

### Integration with CI/CD

Include SBOM generation and scanning in your deployment pipeline:

```yaml
# Example GitHub Actions workflow
- name: Generate SBOM
  run: syft $IMAGE_NAME -o spdx-json > sbom.json

- name: Scan for vulnerabilities
  run: grype sbom:sbom.json --fail-on high
```

### Recommended Tools

1. **Grype** - Fast, accurate vulnerability scanning
2. **Trivy** - Comprehensive security scanner
3. **Docker Scout** - Native Docker vulnerability scanning
4. **Snyk** - Developer-friendly security platform
5. **Aqua Security** - Enterprise-grade container security

---

## SBOM Update Process

When updating component versions:

1. Pull the new Docker image
2. Generate new SBOM: `syft <image>:<tag> -o spdx-json > sbom/<component>-<version>-sbom.spdx.json`
3. Scan for vulnerabilities: `grype sbom:sbom/<component>-<version>-sbom.spdx.json`
4. Compare with previous SBOM to identify new/removed/updated packages
5. Review security scan results
6. Update this reference document with new statistics

---

## Package Statistics Summary

### By Package Manager

Based on analysis of all SBOM files, the primary package managers and ecosystems include:

- **Alpine APK**: System packages (all components use Alpine base)
- **Go Modules**: Application dependencies (Prometheus, Grafana, Loki, Promtail, Alertmanager, Pushgateway)
- **Python PIP**: Python-based components (PVE Exporter, jinja2docker)
- **NPM/Yarn**: Frontend dependencies (Grafana)
- **C/C++ Libraries**: System libraries (syslog-ng)

### Common Dependencies Across Components

Several packages appear across multiple components:
- **musl libc**: Standard C library for Alpine Linux
- **ca-certificates**: SSL/TLS certificate validation
- **tzdata**: Timezone data
- **busybox**: Core utilities
- **Go runtime libraries**: Common Go dependencies

---

## Compliance and Governance

### Standards Compliance

These SBOMs support compliance with:
- **NIST SP 800-218**: SSDF - Secure Software Development Framework
- **Executive Order 14028**: Enhancing Software Supply Chain Security
- **NTIA Minimum Elements**: For Software Bill of Materials
- **SPDX 2.3 Specification**: ISO/IEC 5962:2021

### License Information

All SBOM files include license information for each package. Common licenses found:
- **Apache-2.0**: Prometheus, Grafana, Loki components
- **MIT**: Various npm and Go packages
- **GPL/LGPL**: Some system libraries
- **BSD**: Unix-like system components

To extract full license report:
```bash
for f in sbom/*.json; do
  echo "=== $(basename $f) ==="
  jq '.packages[] | select(.licenseDeclared != "NOASSERTION") | .licenseDeclared' "$f" | sort -u
done
```

---

## Archival and Retention

### File Storage

- SBOM files are stored in the `sbom/` directory
- Files are committed to version control for traceability
- Previous versions maintained for historical comparison

### Retention Policy

- Keep SBOMs for all deployed versions
- Archive SBOMs when decommissioning versions
- Maintain 12-month history minimum for compliance

---

## Verification and Integrity

### Verifying SBOM Files

To verify an SBOM file is valid SPDX:

```bash
# Check JSON validity
cat sbom/prometheus-v3.7.2-sbom.spdx.json | jq . > /dev/null && echo "Valid JSON"

# Verify SPDX version
cat sbom/prometheus-v3.7.2-sbom.spdx.json | jq -r '.spdxVersion'

# Count packages
cat sbom/prometheus-v3.7.2-sbom.spdx.json | jq '.packages | length'
```

### Regenerating SBOMs

To regenerate an SBOM for verification:

```bash
syft <image>:<tag> -o spdx-json > /tmp/verify-sbom.json
diff sbom/<component>-<version>-sbom.spdx.json /tmp/verify-sbom.json
```

Note: SBOMs generated at different times may have minor differences due to timestamp and tool version changes.

---

## Troubleshooting

### Common Issues

**Issue**: SBOM file appears corrupted or invalid JSON

**Solution**: Regenerate with stderr redirected:
```bash
syft <image> -o spdx-json 2>/dev/null > sbom.json
```

**Issue**: Package count seems low

**Solution**: Ensure you're analyzing the complete image, not just a layer:
```bash
syft packages <image> --scope all-layers
```

**Issue**: Missing dependency information

**Solution**: Some packages may not be tracked by package managers. Syft attempts to find these but may miss binaries copied directly into images.

---

## Additional Resources

### Tools
- [Syft](https://github.com/anchore/syft) - SBOM generation
- [Grype](https://github.com/anchore/grype) - Vulnerability scanning
- [SPDX Tools](https://github.com/spdx/tools) - SPDX validation and conversion

### Standards
- [SPDX Specification](https://spdx.github.io/spdx-spec/)
- [NTIA SBOM Guidelines](https://www.ntia.gov/sbom)
- [CISA SBOM Resources](https://www.cisa.gov/sbom)

### Security
- [National Vulnerability Database](https://nvd.nist.gov/)
- [GitHub Security Advisories](https://github.com/advisories)
- [Snyk Vulnerability Database](https://snyk.io/vuln/)

---

## Changelog

### 2025-10-30
- Initial SBOM generation for all components
- Generated using Syft v1.36.0
- SPDX 2.3 format
- Total of 9 components, 2,275 packages documented

---

**Generated:** October 30, 2025  
**Tool:** Syft v1.36.0  
**Format:** SPDX 2.3 JSON  
**Maintained By:** DevOps/Security Team
