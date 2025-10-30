# Release Notes - Component Version Updates (October 2025)

This document details the version upgrades for all components in the hell-compose-monitor stack and their potential impact on users.

## Overview

This release updates all Docker container images to their latest stable versions as of October 2025. Several components have received major version upgrades that include breaking changes and require careful migration planning.

---

## Component Updates

### 🔴 CRITICAL - Prometheus: v2.42.0 → v3.7.2 (MAJOR UPGRADE)

**Breaking Changes:**
- **Native Histogram WAL Incompatibility**: If you used native histograms in v2.42.0, you MUST remove the WAL directory when upgrading to avoid data corruption. This will result in loss of recent metrics data.
- **Default Port Handling**: Prometheus v3 no longer automatically adds default ports (e.g., :443 for HTTPS) to scrape targets. Explicitly specify ports in your target URLs.
- **Feature Flags Removed**: The following feature flags are now default behavior and no longer need to be set:
  - `promql-at-modifier`
  - `promql-negative-offset`
  - `new-service-discovery-manager`
  - `expand-external-labels`
- **Remote Write HTTP/2**: The default for `http_config.enable_http2` in remote_write config is now `false`. Set `http_config.enable_http2: true` if you need HTTP/2 for remote_write.
- **PromQL Regex Changes**: The `.` (dot) pattern now matches newline characters. Review your alerting and query rules.
- **Alertmanager API**: Configurations using `alertmanagers: [api_version: v1]` must be updated to `api_version: v2`.
- **Environment Variables**: References like `${var}` or `$var` in external label values now use current environment variables. Escape `$` with `$$`.

**Migration Steps:**
1. Backup all Prometheus data
2. Stop Prometheus service
3. If using native histograms, delete the WAL directory: `rm -rf <prometheus-data-dir>/wal`
4. Update scrape targets to explicitly define ports where needed
5. Review and update PromQL queries, especially those using regex patterns
6. Update Alertmanager configuration to use API v2
7. Start Prometheus v3.7.2 and monitor logs for warnings

**New Features:**
- Improved native histogram support
- Enhanced OpenTelemetry integration
- Redesigned web UI with better query building
- Automatic GOMEMLIMIT and GOMAXPROCS based on container limits

---

### 🔴 CRITICAL - Grafana: 9.3.6 → 12.2.1 (MAJOR UPGRADE)

**Breaking Changes:**
- **Go Runtime**: Upgraded to Go 1.25.x - custom backends/plugins need compatibility verification
- **Plugin Compatibility**: Many plugins developed for 9.x may not work in 12.x without updates
- **Dashboard Changes**: Some keyboard shortcuts, table panel features, and cross-platform behaviors have changed
- **Authentication**: OAuth passthrough and LDAP URL propagation have been improved - custom auth integrations may need updates
- **Routing**: Login redirects and subpath behaviors have been streamlined - verify custom hosting paths

**Migration Steps:**
1. Backup Grafana database, configuration files, and custom plugins
2. Review plugin compatibility - update or replace incompatible plugins
3. Test in staging environment first
4. Consider incremental upgrade path: 9.3.6 → 10.x → 11.x → 12.x for lower risk
5. Review and test all dashboards after upgrade
6. Verify authentication flows and custom integrations

**Impact:**
- Users may need retraining on new dashboard behaviors
- Custom plugins will likely require updates
- Improved security and performance

**References:**
- [Grafana Changelog](https://github.com/grafana/grafana/blob/main/CHANGELOG.md)
- [Grafana End-of-Life Schedule](https://endoflife.date/grafana)

---

### 🟡 IMPORTANT - Loki & Promtail: 2.7.3 → 3.5.7

**Breaking Changes:**
- **Loki UI Deprecated**: Experimental Loki UI removed from Helm Chart. Migrate to Grafana Loki plugin for UI functionality.
- **Ksonnet Deprecation**: Ksonnet-based configuration options have been deprecated and removed.
- **Go Runtime**: Upgraded to Go 1.24.x - affects custom compiled plugins
- **OTLP Configuration**: New configuration option for dropping OTLP attributes - update operator configurations if using Loki Operator
- **MinIO Warning**: Future deprecation of MinIO as object store planned - prepare alternative storage backend

**New Features:**
- SQLite support for storing delete requests
- Enhanced OTLP attribute handling
- Improved performance and stability

**Migration Steps:**
1. Backup Loki configuration and data
2. If using Helm with `loki.ui.enabled: true`, migrate to Grafana Loki plugin
3. Remove ksonnet-based configuration options
4. Review OTLP configurations if applicable
5. If using MinIO, plan migration to alternative object storage (S3, GCS, Azure Blob)

**Compatibility Notes:**
- Loki and Promtail versions MUST match (both 3.5.7)
- Verify Grafana datasource configuration after upgrade

---

### 🟢 STANDARD - Alertmanager: v0.25.0 → v0.28.1

**Changes:**
- Updated to match Prometheus v3 API requirements
- Bug fixes and performance improvements
- Security patches (see SECURITY_CHECKLIST.md)

**Migration:**
- No breaking changes in configuration
- Compatible with Prometheus v3 API v2 requirement

---

### 🟢 STANDARD - Pushgateway: v1.5.1 → v1.11.1

**Changes:**
- Bug fixes and dependency updates
- Performance improvements
- Enhanced reliability

**Migration:**
- No breaking changes expected
- Drop-in replacement

---

### 🟢 STANDARD - syslog-ng: 4.0.1 → 4.8.0

**Changes:**
- Platform improvements and bug fixes
- Performance enhancements
- Security updates

**Migration:**
- No known breaking changes
- Configuration should remain compatible

---

### 🟢 STANDARD - prometheus-pve-exporter: 2.2.4 → 3.5.5 (MAJOR UPGRADE)

**Changes:**
- Updated dependencies: urllib3, requests, prometheus-client
- Alpine base image updates
- Enhanced Proxmox VE metric collection

**Migration:**
- Verify Proxmox VE API compatibility
- No configuration changes expected

---

### 🟢 STANDARD - jinja2docker: 2.1.6 → 2.1.8

**Changes:**
- Bug fixes and minor improvements

**Migration:**
- No breaking changes

---

## Compatibility Matrix

| Component | Version | Compatible With |
|-----------|---------|-----------------|
| Prometheus | v3.7.2 | Alertmanager v0.28.1, Grafana 12.x |
| Grafana | 12.2.1 | Prometheus v3.x, Loki 3.5.7 |
| Loki | 3.5.7 | Promtail 3.5.7, Grafana 12.x |
| Promtail | 3.5.7 | Loki 3.5.7 |
| Alertmanager | v0.28.1 | Prometheus v3.x |
| Pushgateway | v1.11.1 | Prometheus v3.x |
| syslog-ng | 4.8.0 | Promtail 3.5.7 |
| PVE Exporter | 3.5.5 | Prometheus v3.x |
| jinja2docker | 2.1.8 | N/A (templating) |

---

## Pre-Upgrade Checklist

- [ ] **Backup all data volumes** (Prometheus, Grafana, Loki, Alertmanager)
- [ ] **Export all Grafana dashboards** for safekeeping
- [ ] **Document current configuration** files
- [ ] **Test upgrade in staging environment** first
- [ ] **Review Prometheus WAL** directory handling for native histograms
- [ ] **Update Prometheus scrape configs** to include explicit ports
- [ ] **Update Alertmanager config** to use API v2
- [ ] **Check Grafana plugin compatibility** and update as needed
- [ ] **Verify Loki/Promtail version match** (both must be 3.5.7)
- [ ] **Plan for incremental Grafana upgrade** if skipping major versions
- [ ] **Schedule maintenance window** for upgrade (recommended 2-4 hours)

---

## Post-Upgrade Validation

- [ ] Verify Prometheus is scraping all targets successfully
- [ ] Confirm Alertmanager is receiving and routing alerts
- [ ] Test Grafana dashboards and verify all panels load correctly
- [ ] Validate Loki log ingestion from Promtail
- [ ] Check syslog-ng log forwarding to Promtail
- [ ] Verify PVE Exporter metrics collection
- [ ] Review logs for any warnings or errors
- [ ] Test alert routing and notifications

---

## Rollback Plan

If issues occur during upgrade:

1. **Stop all services**: `docker-compose down`
2. **Restore data volumes** from backup
3. **Revert docker-compose.yml** to previous version: `git checkout HEAD~1 docker-compose.yml`
4. **Start services**: `docker-compose up -d`
5. **Verify functionality**

---

## Support and Resources

- [Prometheus Migration Guide](https://prometheus.io/docs/prometheus/latest/migration/)
- [Grafana Changelog](https://github.com/grafana/grafana/blob/main/CHANGELOG.md)
- [Loki Changelog](https://github.com/grafana/loki/blob/main/CHANGELOG.md)
- [Alertmanager Releases](https://github.com/prometheus/alertmanager/releases)

---

**Last Updated:** October 30, 2025
