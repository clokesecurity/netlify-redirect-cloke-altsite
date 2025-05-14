# netlify-redirect-cloke-altsite

Below is a step‑by‑step field‑guide for parking an alternative domain on **Netlify** and having it issue a clean, SEO‑friendly **301** to your canonical site—all while keeping the nameservers at Namecheap and paying nothing beyond Netlify’s Free tier.

---

## Quick‑look summary

Create a “redirect‑only” Netlify site that contains nothing but a `_redirects` file instructing the edge to send every request to `https://your‑main.com`. Add your alt domain to the site, then point DNS at Netlify (CNAME for `www`, ALIAS/ANAME or an A‑record for the apex). Netlify provisions a free Let’s Encrypt cert automatically, so both HTTP and HTTPS visitors get an instant 301. You stay on Namecheap DNS; no Cloudflare required. ([Netlify][1], [Netlify][2], [Netlify][3])

---

## 1 · Set up a “redirect‑only” repo

1. **Create an empty folder/repo** (`netlify‑redirect‑altsite`).

2. Inside, add a plaintext file named **`_redirects`**:

   ```text
   /*   https://example.com/:splat   301
   ```

   * `/*` = match every path.
   * `:splat` carries over whatever path or query string the visitor used. ([Netlify][1])

3. Commit and push to GitHub/GitLab/Bitbucket, or be lazy and drag‑drop the folder onto the Netlify UI.

> **Alternative:** put the rule in `netlify.toml` if you already use that file – hierarchy and syntax are identical. ([DEV Community][4])

---

## 2 · Deploy the site on Netlify

1. In the Netlify dashboard, click **“Add new site → Import from Git”** (or **“Deploy manually”** if you used drag‑and‑drop).
2. When asked for a build command, leave it blank; for the publish directory use `/` (root). Netlify will see the `_redirects` file and finish the deploy in seconds. ([Netlify][5])

---

## 3 · Attach your alt domain

Open the site → **Domain management → Add domain → Domain you already own** and enter `alt‑domain.tld`. Netlify verifies ownership via DNS and then marks the domain “Awaiting external DNS”. ([Netlify][3])

---

## 4 · Point DNS at Netlify (stay on Namecheap)

| Case                                    | Record to add in Namecheap                     | Points to                       | Notes                                                                                        |
| --------------------------------------- | ---------------------------------------------- | ------------------------------- | -------------------------------------------------------------------------------------------- |
| **`www` or any sub‑domain**             | **CNAME** `www.alt-domain.tld`                 | `your‑site.netlify.app`         | Sub‑domains can always use CNAME. ([Netlify][3])                                             |
| **Apex/root domain** (`alt-domain.tld`) | **Preferred**: ALIAS / ANAME / flattened CNAME | `apex-loadbalancer.netlify.com` | Works only if DNS provider supports those pseudo‑records. ([Netlify][3])                     |
|                                         | **Fallback**: **A** record                     | `75.2.60.5`                     | Netlify’s load‑balancer IP. (A second IP is returned via DNS for redundancy.) ([Netlify][3]) |

> **Propagation tip:** both Netlify and Namecheap caches can take up to 24 h; running `dig +short alt-domain.tld` until it shows the Netlify value confirms the switch. ([Netlify][3])

---

## 5 · Automatic HTTPS and the 301

* As soon as the DNS change is detected, Netlify provisions a **Let’s Encrypt certificate** for the domain and renews it automatically. ([Netlify][2], [Netlify][6])
* Your `_redirects` rule returns **`301 Moved Permanently`** by default (explicit `301` in the file is optional but self‑documenting). ([Netlify][7])
* The redirect happens at the CDN edge, so latency is minimal. ([CSS-Tricks][8])

---

## 6 · Free‑tier limits and cost

| Limit                     | Free tier allowance                                                                            |
| ------------------------- | ---------------------------------------------------------------------------------------------- |
| Monthly requests          | 100 k site‑wide (beyond that, US \$0.30 / 100 k) ([Netlify][9])                                |
| Bandwidth                 | 100 GB per month (then \$20 / 100 GB pack) ([Netlify][9], [Netlify Support Forums][10])        |
| Redirect rules per deploy | \~10 k before build fails (wildcards recommended) ([Netlify][1], [Netlify Support Forums][11]) |

For a parked domain with only one wildcard rule, you are effectively operating at \$0.

---

## 7 · Validation checklist

1. `curl -I https://alt-domain.tld/path?q=1` → expect `HTTP/2 301` and `Location: https://example.com/path?q=1`.
2. Visit in a browser: no SSL warning (Netlify cert in place).
3. Google Search Console → **Change of Address** from alt domain to canonical to speed up de‑indexing.

---

## 8 · Why choose Netlify over Namecheap’s built‑in redirect?

* **HTTPS support:** Namecheap’s forwarder has no cert for your domain, so HTTPS breaks; Netlify issues one automatically. ([Netlify][2])
* **Edge performance:** Redirect is served from Netlify’s global CDN Pops. ([CSS-Tricks][8])
* **Git‑ops workflow:** Rule lives in source control; future updates are one commit. ([CSS-Tricks][12])

---

### That’s all there is: a one‑line `_redirects` file, a quick deploy, and two DNS records. You’ve now got a zero‑maintenance, SSL‑clean 301 for every visit hitting your alternative domain—and you never had to move the domain off Namecheap or pay Cloudflare a cent.

[1]: https://docs.netlify.com/routing/redirects/?utm_source=chatgpt.com "Redirects and rewrites | Netlify Docs"
[2]: https://docs.netlify.com/domains/secure-domains-with-https/https-ssl/?utm_source=chatgpt.com "HTTPS (SSL) | Netlify Docs"
[3]: https://docs.netlify.com/domains/configure-domains/configure-external-dns/ "Configure external DNS for a custom domain | Netlify Docs"
[4]: https://dev.to/mlaposta/how-to-redirect-urls-with-netlify-22ah?utm_source=chatgpt.com "How to Redirect URLs with Netlify - DEV Community"
[5]: https://www.netlify.com/blog/2019/01/16/redirect-rules-for-all-how-to-configure-redirects-for-your-static-site/?utm_source=chatgpt.com "Redirect Rules for All; How to configure redirects for your ... - Netlify"
[6]: https://www.netlify.com/blog/2016/01/15/free-ssl-on-custom-domains/?utm_source=chatgpt.com "A World’s First. Free SSL with Let’s Encrypt | Netlify"
[7]: https://docs.netlify.com/routing/redirects/redirect-options/?utm_source=chatgpt.com "Redirect options | Netlify Docs"
[8]: https://css-tricks.com/static-first-pre-generated-jamstack-sites-with-serverless-rendering-as-a-fallback/?utm_source=chatgpt.com "Static First: Pre-Generated JAMstack Sites with Serverless ... - CSS-Tricks"
[9]: https://www.netlify.com/pricing/?utm_source=chatgpt.com "Pricing and Plans - Netlify"
[10]: https://answers.netlify.com/t/limiting-bandwidth-traffic-to-netlify-on-starter-tier-plan/18163?utm_source=chatgpt.com "Limiting bandwidth/traffic to netlify on starter tier plan"
[11]: https://answers.netlify.com/t/max-number-of-redirects/5095?utm_source=chatgpt.com "Max number of redirects? - Netlify Support Forums"
[12]: https://css-tricks.com/fast-static-sites-with-netlify-and-anymod/?utm_source=chatgpt.com "Fast Static Sites With Netlify And AnyMod | CSS-Tricks"
