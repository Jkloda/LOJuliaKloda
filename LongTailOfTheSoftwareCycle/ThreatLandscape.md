### Threat Landscape - Just Pick

#### LLM Model

- Prompt Injection – prompt which can expose data or manipulate leakage
  Mitigation: Input sanitization, output filtering
- Bad Recommendations – AI suggest inappropriate suggestions, which can offend users.
  Mitigation: Add filters which block wrong outputs

#### MySQL Database

- SQL injection – using SQL command in text field, which allow taking control oof database
  Mitigation: validate user inputs
- Database overload – too many requests crash the database,
  Mitigation: Use cashing and limit repeated requests
- Weak permissions – too big access (e.g. admin panel), which can allow to view sensitive data.
  Mitigation: Review permissions regularly, apply restrictions

#### API

- Broken Authentication – access doesn’t work properly, hackers and users can get access into admin areas.
  Mitigation: Use proven auth libraries, test the access
- Token theft – API tokens stolen, which allow attackers
  Mitigation: Use secure cookies
- Weak login/register system – user can login with wrong email, reused data, make it easy to attack.
  Mitigation: Enforce strong passwords
- Too Many Requests – overload attack, user sending lots of requests, which slow down the system.
  Mitigation: Add request limit, timeout

#### Infrastructure

- Insider Threat – team member can access system in purpose to delete or leak sensitive data.
  Mitigation: team supposed to log action, limit access
- Bad dependencies – project is using open-source libraries which can have bugs or be infected with malware, hackers can steal data or run harmful code.
  Mitigation: scan vulnerabilities, update dependencies, pip-audit to keep code safe
