# Corefig: 10 Lightweight Improvement Ideas

These ideas are intentionally low-complexity and low-resource so they can be delivered incrementally without changing Corefig’s core workflow.

1. **Add a first-run “Quick Tasks” panel**
   Show a short list of the most common actions (rename server, configure network, enable WinRM, run updates) when Corefig starts. This gives new admins an obvious starting point and reduces hunting through modules. The value is faster onboarding with minimal UI work because it can reuse existing commands.

2. **Introduce a global search box for modules/actions**
   Add a simple text filter that narrows visible sections or jumps directly to matching tasks. This makes Corefig feel much faster to navigate, especially for infrequent users who forget where features live. It improves usability without requiring any backend changes.

3. **Display a clear “pending restart required” banner**
   Track actions that usually require reboot and show one persistent notice until the restart happens. This helps prevent confusion when settings do not appear applied immediately. The change improves operational reliability with straightforward state tracking.

4. **Provide copy-to-clipboard for logs and error details**
   Add a button near error dialogs/log views to copy full technical details in one click. This reduces support time and helps admins share precise diagnostics quickly. It is low effort and has an immediate supportability payoff.

5. **Add a “Safe defaults” mode for risky settings**
   Offer a toggle that preselects conservative options (for firewall, remote access, and service exposure) while still allowing overrides. This lowers accidental misconfiguration risk for less experienced operators. It adds guardrails without taking control away from advanced users.

6. **Create a simple built-in “health summary” page**
   Show a compact status snapshot (domain joined, RDP state, WinRM state, update mode, firewall profile, IP mode). This gives admins quick situational awareness before making changes. Most data already exists, so this is mainly presentation work with high practical value.

7. **Standardize success/error messages across modules**
   Use a consistent message format: what changed, current state, and next action if needed. Consistent wording improves confidence and reduces interpretation errors. This is a low-risk refinement that improves user trust and training.

8. **Add “Export configuration snapshot” to a text/JSON file**
   Let admins export current key settings for documentation, handoff, and before/after comparisons. This helps change control and troubleshooting without requiring full backup tooling. Implementation can start with a small set of high-value settings and expand later.

9. **Improve accessibility with better keyboard flow and focus order**
   Ensure tabs, buttons, and form fields follow logical keyboard navigation order and have clearer labels. This helps both accessibility compliance and power users who prefer keyboard workflows. It brings broad UX gains with mostly UI-layer adjustments.

10. **Include inline help tooltips with “why this matters” text**
    Add brief explanations next to sensitive settings (e.g., WinRM, firewall, updates) so users understand impact before applying. This reduces mistakes and dependency on external docs. It is inexpensive content work that can dramatically improve decision quality.
