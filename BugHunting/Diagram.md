```
+-----------------------------+        Your Target as a Bug Hunter
| 1. END USER / ATTACKER      |  ← You start here (browser, mobile app, API client)
|   (Browser, Mobile App,     |
|    curl, Burp, etc.)        |
+-----------------------------+
          ↓  HTTPS / TLS
+-----------------------------+
| 2. FRONTEND                 |  ← Most bug bounty scope starts here
|   (React, Angular, Vue,     |    XSS, CSRF, DOM bugs, open redirects,
|    static JS/CSS/HTML)      |    business logic flaws, IDOR, etc.
+-----------------------------+
          ↓  HTTP(S) + JSON/XML/API calls
+-----------------------------+
| 3. BACKEND SERVERS          |  ← The real money in bug bounties
|   (Node.js, Python/Django,  |    SSRF, SQLi, RCE, auth bypass,
|    Java Spring, Go, PHP,    |    deserialization, mass assignment,
|    Ruby on Rails, etc.)     |    rate-limit bypass, etc.
+-----------------------------+
          ↓  Local sockets / DB drivers
+-----------------------------+
| 4. USERSPACE SERVICES       |
|   (Redis, Memcached,        |
|    PostgreSQL/MySQL,        |
|    Elasticsearch, RabbitMQ, |
|    Nginx, cron jobs, etc.)  |
+-----------------------------+
          ↓  System Calls (syscall gate)
┌────────────────────────────────────────────────────┐
│                  PRIVILEGE BOUNDARY               │ ← The most valuable boundary to cross
│  Ring 3 (User)  ←─────── syscall / sysenter ──────→ Ring 0 (Kernel)     │
└────────────────────────────────────────────────────┘
          ↓
+-----------------------------+
| 5. THE KERNEL               |  ← Local privilege escalation lives here
|   (Linux, Windows NT,       |    Kernel drivers, use-after-free,
|    macOS XNU, etc.)         |    race conditions, Dirty COW-style,
|   • System-call handlers    |    ret2dir, stack pivoting, etc.
|   • Device drivers          |
|   • Memory management (SLUB)|
|   • Filesystems, network    |
|     stack, scheduler        |
+-----------------------------+
          ↓  CPU rings & page tables
+-----------------------------+
| 6. HARDWARE / CPU           |
|   Ring 0  → Kernel          |
|   Ring -1 → Intel ME / AMD PSP / SMM           |
|   Ring -2 → Hypervisor (VM Escape)             |
|   Ring -3 → UEFI / Secure Boot / Firmware     |
+-----------------------------+
```