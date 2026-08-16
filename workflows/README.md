# Raven Workflows

This directory contains workflow YAML files that automate multi-step penetration testing tasks. Workflows allow you to chain together multiple Raven modules with intelligent conditional execution.

## What are Workflows?

Workflows are YAML files that define automated testing sequences. Each workflow:
- Chains together multiple Raven modules
- Defines dependencies between steps
- Uses conditional execution to skip unnecessary steps
- Automatically passes data between modules
- Provides a repeatable, documented testing process

## Using Workflows

Execute a workflow from the Raven CLI:

```bash
raven workflow run <workflow-name>
```

Or from the web UI:
1. Navigate to the Workflows page
2. Select a workflow
3. Click "Run Workflow"

## Included Workflow Templates

17 templates ship with Raven. Fork any of them as a starting point.

- `api-security.yaml`
- `bug-bounty.yaml`
- `credential-hunting.yaml`
- `dependency-example.yaml`
- `external-perimeter.yaml`
- `full_external.yaml`
- `network-infrastructure.yaml`
- `osint-comprehensive.yaml`
- `osint_recon.yaml`
- `owasp-wstg.yaml`
- `ptes.yaml`
- `quick-recon.yaml`
- `quick_scan.yaml`
- `recon_summary.yaml`
- `vuln_assessment.yaml`
- `web-app-assessment.yaml`
- `web_app_testing.yaml`

## Creating Custom Workflows

You can create your own custom workflow YAML files in this directory. Raven will automatically detect and load them.

### Basic Workflow Structure

```yaml
name: My Custom Workflow
description: Brief description of what this workflow does
enabled: true

steps:
  - name: step_identifier
    module: module_name
    description: What this step does
    params: {}
```

### Step Parameters

Each step supports the following fields:

- **name** (required): Unique identifier for this step
- **module** (required): Name of the Raven module to execute
- **description** (required): Human-readable description
- **params** (optional): Dictionary of module-specific parameters
- **condition** (optional): Conditional execution based on previous results
- **depends_on** (optional): List of step names that must complete first

### Conditional Execution

Steps can be conditionally executed based on the results of previous steps:

```yaml
steps:
  - name: subdomain_enum
    module: subfinder
    description: Enumerate subdomains
    params: {}

  - name: port_scan
    module: nmap
    description: Scan discovered hosts
    condition: hosts_found          # Only run if hosts were discovered
    depends_on:
      - subdomain_enum
    params:
      scan_type: fast
```

#### Available Conditions

- `subdomains_found` - At least one subdomain was discovered
- `hosts_found` - At least one host IP was discovered
- `web_services_found` - At least one web service (HTTP/HTTPS) was found
- `ssl_services_found` - At least one SSL/TLS service was found
- `vulnerabilities_found` - At least one vulnerability was identified
- `emails_found` - At least one email address was harvested
- `breaches_found` - At least one credential breach was found

### Module Parameters

Different modules accept different parameters. Here are some common examples:

#### Nmap Scanning
```yaml
- name: port_scan
  module: nmap
  params:
    scan_type: full              # Options: quick, fast, full
    top_ports: 1000              # Number of top ports to scan
    service_detection: true      # Enable service version detection
    os_detection: true           # Enable OS detection
```

#### Nuclei Vulnerability Scanning
```yaml
- name: vuln_scan
  module: nuclei
  params:
    templates: cves,exposures    # Template categories to use
    severity: critical,high      # Only scan for these severities
```

#### Directory Bruteforcing
```yaml
- name: dir_bruteforce
  module: ffuf
  params:
    wordlist: common             # Options: common, medium, large
    threads: 50                  # Number of concurrent threads
```

### Example: Custom Bug Bounty Workflow

```yaml
name: Bug Bounty Quick Assessment
description: Fast vulnerability discovery for bug bounty programs
enabled: true

steps:
  - name: subdomain_enum
    module: subfinder
    description: Discover all subdomains
    params:
      timeout: 600

  - name: quick_scan
    module: nmap
    description: Fast port scan
    condition: hosts_found
    depends_on:
      - subdomain_enum
    params:
      scan_type: quick
      top_ports: 1000

  - name: screenshots
    module: gowitness
    description: Capture screenshots
    condition: web_services_found
    depends_on:
      - quick_scan
    params:
      timeout: 10

  - name: web_tech
    module: fingerprinting
    description: Identify technologies
    condition: web_services_found
    depends_on:
      - quick_scan
    params: {}

  - name: high_vuln_scan
    module: nuclei
    description: Scan for critical vulnerabilities
    condition: web_services_found
    depends_on:
      - quick_scan
    params:
      severity: critical,high
      templates: cves,exposures,vulnerabilities

  - name: github_leaks
    module: github_dork_scope
    description: Search for leaked secrets
    condition: subdomains_found
    depends_on:
      - subdomain_enum
    params: {}
```

## Best Practices

1. **Start with reconnaissance** - Begin workflows with subdomain enumeration or host discovery
2. **Use conditions wisely** - Skip unnecessary steps when prerequisites aren't met
3. **Define dependencies** - Ensure steps run in the correct order
4. **Keep descriptions clear** - Help users understand what each step does
5. **Test workflows thoroughly** - Run workflows on test targets before production use
6. **Document custom parameters** - Add comments explaining non-obvious settings

## Advanced Features

### Parallel Execution

Steps without dependencies can run in parallel automatically. To ensure steps run sequentially, use the `depends_on` field.

### Skipping Steps

If a condition isn't met, the step is skipped and logged. Subsequent steps that depend on the skipped step will also be skipped.

### Error Handling

If a step fails:
- The error is logged
- Subsequent dependent steps are skipped
- Independent steps continue executing

## Available Modules

Common modules you can use in workflows:

**Reconnaissance:**
- `subdomain_enum` - Subdomain enumeration
- `nmap_scope` - Port scanning
- `gowitness_scope` - Screenshot capture
- `wappalyzer_scope` - Technology detection

**OSINT:**
- `email_harvest_scope` - Email harvesting
- `github_dork_scope` - GitHub secret hunting
- `wayback_scope` - Wayback Machine enumeration
- `shodan_host_scope` - Shodan host lookups
- `dehashed_scope` - Breach data checking

**Web Testing:**
- `wafw00f_scope` - WAF detection
- `feroxbuster_scope` - Directory bruteforcing
- `git_exposure_scope` - Git repository exposure
- `api_discovery_scope` - API endpoint discovery
- `nuclei_scope` - Vulnerability scanning

**Exploitation:**
- `searchsploit_scope` - Exploit database search

For a complete list of available modules, run:
```bash
raven modules list
```

## Workflow Locations

Raven looks for workflows in these locations:
1. Engagement-specific workflows: `<engagement-dir>/workflows/`
2. Global workflows: `~/.raven/workflows/`
3. System workflows: `/opt/raven/workflows/`

Engagement-specific workflows (like those in this directory) take precedence over global workflows.

## Tips for Custom Workflows

- **Copy and modify** existing workflow templates as a starting point
- **Chain OSINT modules** together for comprehensive intelligence gathering
- **Combine recon and testing** for end-to-end assessments
- **Create role-specific workflows** (e.g., "web-app-only", "network-only")
- **Build time-boxed workflows** for quick assessments with timeouts
- **Document your workflows** with clear descriptions for team collaboration

## Need Help?

- Check existing workflow files for examples
- Review module documentation: `raven modules info <module-name>`
- Test workflows on controlled targets first
- Report issues or request features on the Raven GitHub repository

---

**Note:** Workflow execution respects your API keys and tool configurations. Ensure all required API keys (Shodan, DeHashed, GitHub, etc.) are configured in Settings before running workflows that use them.
