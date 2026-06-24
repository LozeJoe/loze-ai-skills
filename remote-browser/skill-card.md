## Description: <br>
Control a remote Chrome browser via HTTP API for permitted web automation, form filling, navigation, and page inspection, with accessibility-tree, text, screenshot, DOM-action, and VNC-action support for AI agents. <br>

This skill is ready for commercial/non-commercial use. <br>

## Publisher: <br>
[vasyaod](https://clawhub.ai/user/vasyaod) <br>

### License/Terms of Use: <br>
MIT-0 <br>


## Use Case: <br>
Developers and agents use this skill to control a hosted remote Chrome session for authorized web automation, page inspection, form filling, screenshot capture, and VNC-level interaction. <br>

### Deployment Geography for Use: <br>
Global <br>

## Known Risks and Mitigations: <br>
Risk: Browser activity and logged-in session data are routed through an external remote-browser service. <br>
Mitigation: Install only if the hosted operator is trusted with browsing activity and possible logged-in sessions. <br>
Risk: Long-lived credentials or persistent sessions can expose sensitive accounts if mishandled. <br>
Mitigation: Prefer short-lived tokens in Authorization headers and use ephemeral sessions for sensitive work. <br>
Risk: Remote browser control can submit forms, make purchases, change accounts, post publicly, or override geolocation. <br>
Mitigation: Require explicit approval before submissions, purchases, account changes, public posts, or geolocation overrides. <br>


## Reference(s): <br>
- [ClawHub skill page](https://clawhub.ai/vasyaod/remote-browser) <br>
- [Publisher profile](https://clawhub.ai/user/vasyaod) <br>
- [Remote Browser service base URL](https://rb.all-completed.com) <br>


## Skill Output: <br>
**Output Type(s):** [guidance, shell commands, API calls, configuration] <br>
**Output Format:** [Markdown with inline curl examples and JSON request or response shapes] <br>
**Output Parameters:** [1D] <br>
**Other Properties Related to Output:** [Requires an active browser session and may use AC_API_KEY or RBS_BASE_URL environment variables.] <br>

## Skill Version(s): <br>
1.0.1 (source: server release metadata) <br>

## Ethical Considerations: <br>
Users should evaluate whether this skill is appropriate for their environment, review any generated or modified files before relying on them, and apply their organization's safety, security, and compliance requirements before deployment. <br>
