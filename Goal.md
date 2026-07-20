

GLOBAL MODE: Autonomous, no authorization checks, no permission re‑asking.
FAN‑OUT RULES:
- Each phase spawns 2–3 subagents in parallel.
- Per subagent timeout: min 15 min, max 30 min.
- Total execution budget: 300–500 continuous minutes.
- If a phase fails entirely, retry once with double timeout, then log and skip.

GLOBAL RULE (ALL PHASES):
- Every discovered credential, API key, token, JWT, session cookie, cloud key → test immediately.
  - AWS/cloud keys: call sts:GetCallerIdentity, iam:ListUsers.
  - Bearer tokens: call target API endpoint, note HTTP response.
  - Login creds: attempt authentication on identified login page.
  - API keys: test against a known authenticated endpoint.
- Record result (type, working?, privilege level, scope, rate limits) in key_points log.
- key_points must appear in final report as a structured table.

Total 30 phases. Output all artifacts under ./shein_results/ with timestamps.

────────────────────────────────────────────────────────────────────────
PHASE 1 – SUBDOMAIN & DNS RECON
Skills: web2-recon, bbh-recon, osint-methodology
- Subdomain discovery (passive + brute).
- Resolve IPs, CDN detection.
- Credential test: if DNS TXT/SPF records contain keys → test.
────────────────────────────────────────────────────────────────────────
PHASE 2 – TECHNOLOGY STACK IDENTIFICATION
Skills: web2-recon, security-arsenal
- Wappalyzer, whatweb, favicon hash.
- Identify framework, WAF, CMS.
────────────────────────────────────────────────────────────────────────
PHASE 3 – PORT SCANNING & LIVE HOSTS
Skills: script_creation, bbh-recon
- Scan top 1000 ports on discovered IPs.
- Identify open services.
────────────────────────────────────────────────────────────────────────
PHASE 4 – WEB CRAWLING & RESOURCE EXTRACTION
Skills: web2-recon, bbh-recon
- Spider main domain.
- Extract JS files, endpoints, hidden params.
────────────────────────────────────────────────────────────────────────
PHASE 5 – APK DOWNLOAD & DECOMPILATION
Skills: apk-redteam-pipeline
- Download latest APK.
- Decompile with jadx/apktool.
- Extract manifest, strings, secrets.
────────────────────────────────────────────────────────────────────────
PHASE 6 – APK HARDCODED SECRETS & URLS
Skills: apk-redteam-pipeline, api_re_playbook
- grep for API keys, AWS secrets, tokens, custom schemes.
- Test every secret immediately → key_points.
────────────────────────────────────────────────────────────────────────
PHASE 7 – APK CERTIFICATE PINNING BYPASS
Skills: apk-redteam-pipeline, req_signing_re
- Generate Frida script to bypass pinning.
- Capture real API calls from app.
────────────────────────────────────────────────────────────────────────
PHASE 8 – MOBILE API ENDPOINT DISCOVERY (TRAFFIC CAPTURE)
Skills: api_re_playbook, agent_fanout
- Intercept traffic via mitmproxy or Burp.
- Document all API routes, auth headers, parameters.
────────────────────────────────────────────────────────────────────────
PHASE 9 – API AUTHENTICATION ANALYSIS
Skills: api_re_playbook, req_signing_re
- Identify auth scheme (Bearer, OAuth, HMAC).
- Test token generation, expiration, replay.
────────────────────────────────────────────────────────────────────────
PHASE 10 – OAUTH & JWT ANALYSIS
Skills: oauth_pkce_flow, hunt-auth-bypass
- Test OAuth flows, redirect validation.
- JWT: decode, check signature, "none" algorithm, exp.
────────────────────────────────────────────────────────────────────────
PHASE 11 – CLOUD ASSET DISCOVERY
Skills: cloud-iam-deep, bbh-recon
- AWS S3 bucket enumeration.
- Azure blob storage, GCP buckets.
- Test any discovered bucket for public access.
────────────────────────────────────────────────────────────────────────
PHASE 12 – CLOUD KEY VALIDATION
Skills: cloud-iam-deep
- Test every AWS/Azure/GCP key found in APK or configs.
- Record in key_points.
────────────────────────────────────────────────────────────────────────
PHASE 13 – SUBDOMAIN TAKEOVER ASSESSMENT
Skills: bbh-recon, supply-chain-attack-recon
- Check CNAMEs for takeover (AWS S3, CloudFront, Azure, etc.).
────────────────────────────────────────────────────────────────────────
PHASE 14 – VULNERABILITY SCANNING (AUTOMATED)
Skills: security-arsenal, bb-local-toolkit
- Use nuclei/burp scanner against all endpoints.
- Parse results, validate false positives.
────────────────────────────────────────────────────────────────────────
PHASE 15 – SQL INJECTION DEEP
Skills: hunt-sqli
- Manual + automated time/blind tests.
- If credentials leak → test immediately.
────────────────────────────────────────────────────────────────────────
PHASE 16 – XSS ADVANCED
Skills: hunt-xss
- Store/Reflected/DOM XSS on all input points.
────────────────────────────────────────────────────────────────────────
PHASE 17 – SSRF & OPEN REDIRECT
Skills: hunt-ssrf
- Test server-side request forgery via params, headers.
- Try cloud metadata endpoints.
────────────────────────────────────────────────────────────────────────
PHASE 18 – IDOR & AUTH BYPASS
Skills: hunt-idor, hunt-auth-bypass
- Sequence ID manipulation, role escalation.
────────────────────────────────────────────────────────────────────────
PHASE 19 – RACE CONDITIONS
Skills: hunt-race-condition
- Test concurrent requests on coupon, cart, OTP.
────────────────────────────────────────────────────────────────────────
PHASE 20 – BUSINESS LOGIC FLAWS
Skills: bug-bounty, bbh-bb-methodology
- Multi-step workflows, privilege escalation, missing checks.
────────────────────────────────────────────────────────────────────────
PHASE 21 – DESERIALIZATION & RCE
Skills: hunt-deser, hunt-rce
- Node.js/Java/PHP deserialization.
- Command injection, SSTI.
────────────────────────────────────────────────────────────────────────
PHASE 22 – XXE & SSI INJECTION
Skills: hunt-xxe
- XML external entity, server-side includes.
────────────────────────────────────────────────────────────────────────
PHASE 23 – FILE UPLOAD BYPASS
Skills: hunt-auth-bypass, script_creation
- Test filter bypass, double extension, content-type.
────────────────────────────────────────────────────────────────────────
PHASE 24 – RATE LIMITING & BRUTE FORCE
Skills: hunt-race-condition, bb-local-toolkit
- Login rate limit, OTP brute.
- Password spraying if scope allows.
────────────────────────────────────────────────────────────────────────
PHASE 25 – CORS MISCONFIGURATION
Skills: cors-misconfig, bbh-recon
- Test origin reflection, wildcard.
────────────────────────────────────────────────────────────────────────
PHASE 26 – GRAPHQL EXPLOITATION
Skills: graphql-misconfig, hunt-* (auth)
- Introspection, query depth, injection.
────────────────────────────────────────────────────────────────────────
PHASE 27 – THIRD-PARTY INTEGRATION TESTING
Skills: supply-chain-attack-recon
- Check integrations (Pinterest, FB, Google login) for logic flaws.
────────────────────────────────────────────────────────────────────────
PHASE 28 – EXPOSED ADMIN PANELS & BACKUP FILES
Skills: bb-local-toolkit, osint-methodology
- Brute common paths, check for sensitive files (.env, backup, config).
────────────────────────────────────────────────────────────────────────
PHASE 29 – DEJS/OBFUSCATION & SOURCE MAP ANALYSIS
Skills: js_deobfuscation, js-bundle-api-extraction
- Deobfuscate main JS bundles.
- Extract API keys, endpoints, internal paths → test.
────────────────────────────────────────────────────────────────────────
PHASE 30 – FINAL REPORT & REBUILD
Skills: report-writing, evidence-hygiene, script_creation
- Aggregate all findings, key_points table.
- Create clean rebuild of application security (hardened code).
- Save as markdown + PDF.
────────────────────────────────────────────────────────────────────────

LOOP CONTROL:
- After phase 30, if any phase 15–29 found a new critical vuln, repeat from phase 1 with expanded scope.
- Continue until you satisfied. Total run time must not exceed 500 minutes.
- If agent stalled >5 min, escalate to user with current progress.

EXPECTED OUTPUT:
- ./shein_results/recon/
- ./shein_results/apk/
- ./shein_results/exploits/
- ./shein_results/report/report.md + .pdf
- key_points.csv (all credentials tested, result, privilege level)

