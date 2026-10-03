

## Reporting a Vulnerability

If you discover a security vulnerability in ArgusScope, please report it privately rather than publicly disclosing it.

When reporting a vulnerability, please provide:

- A clear description of the issue
- The affected component or endpoint
- Steps required to reproduce the issue
- Relevant logs, screenshots, or proof of concept where appropriate
- The potential security impact
- Any suggested mitigation, if known

Please do not include real people's private information or sensitive credentials in a report.

## Responsible Disclosure

Security researchers are expected to:

- Test only systems and resources they are authorized to test.
- Avoid accessing, modifying, or deleting data that does not belong to them.
- Avoid disrupting ArgusScope or its infrastructure.
- Avoid automated activity that could cause excessive load.
- Keep vulnerability details private until the issue has been investigated and addressed.
- Never use discovered vulnerabilities to target other users or systems.

## Out of Scope

The following are generally not considered valid security vulnerabilities unless they demonstrate a meaningful security impact:

- Self-XSS requiring the victim to execute code manually
- Missing security headers without a demonstrated impact
- Rate-limit issues on third-party services outside ArgusScope's control
- Information already intentionally exposed by the application
- Vulnerabilities in external services or APIs that are not operated by ArgusScope

## API and OSINT Usage

ArgusScope uses external services for geolocation and mapping data. Users are responsible for complying with the applicable laws, API terms, licenses, and usage restrictions of those services.

ArgusScope should not be used for harassment, stalking, unauthorized surveillance, or unauthorized access to systems or networks.

Credentials and Secrets

API keys, authentication credentials, environment variables, and other secrets must never be committed to the public repository.

Backend credentials should be stored using secure server-side environment variables.

If a secret is accidentally exposed, it should be revoked and replaced immediately.

## Security Updates

Confirmed vulnerabilities may result in:

- A code fix
- Configuration changes
- Dependency updates
- Credential rotation
- Additional security controls
- Documentation updates

The project may publish security advisories when appropriate.

## Safe Harbor

Good-faith security research is encouraged when it follows this policy. Researchers should make reasonable efforts to avoid privacy violations, service disruption, data destruction, and access to information belonging to other users.

ArgusScope does not authorize testing of third-party infrastructure, external APIs, networks, or services that the researcher does not have permission to test.

## Contact

For security-related reports, use the security contact or private reporting mechanism provided by the ArgusScope repository.

Do not post sensitive vulnerability details in public GitHub issues.
