# PHP & WordPress – Interview Notes

> **Target**: Senior Engineers, Backend / CMS | **Level**: Amazon, Flipkart, Razorpay

---

## 1. Overview

**What it is**: PHP is a server-side scripting language for the web. WordPress is a CMS built on PHP (themes, plugins, REST API) used for sites and headless content.

**Why companies use it**: PHP: huge hosting support, mature ecosystem, fast to build. WordPress: quick sites, themes/plugins, REST API for headless; powers a large share of the web.

**When NOT to use it**: When you need strong typing and modern tooling out of the box (consider Node/Python); when building real-time or very high-scale APIs from scratch; when team has no PHP experience and project is greenfield.

---

## 2. Core Concepts (From PDF)

### PHP Basics

| Concept | Definition | Simple Explanation | What Interviewers Test | Common Follow-up |
|---------|------------|--------------------|-------------------------|------------------|
| PHP | Server-side scripting language | Runs on server; outputs HTML/JSON | PHP vs Node/Python | |
| Variables | $name | Typed at runtime | Type juggling | |
| Arrays | indexed, associative | List or key-value | Array functions | |
| OOP in PHP | class, object, properties, methods | Encapsulation, inheritance, polymorphism | Access modifiers | |
| public/private/protected | Visibility | Who can access | When to use | |
| __construct() | Constructor | Runs on new | Object lifecycle | |
| Interfaces & Abstract | Contracts | Implement or extend | When to use | |
| Traits | Code reuse | Multiple "inheritance" | vs inheritance | |
| Namespaces | Avoid name collision | Organize classes | PSR-4 | |
| Composer | Dependency manager | composer.json, autoload | Modern PHP | |
| try/catch | Exception handling | Catch and handle | Error vs Exception | |
| Sessions & Cookies | User state | Server-side vs client | Auth | |

### Security (PHP)

| Concept | Definition | Simple Explanation | What Interviewers Test | Common Follow-up |
|---------|------------|--------------------|-------------------------|------------------|
| SQL Injection | Malicious SQL via input | Parameterized queries | Prevention | Prepared statements |
| XSS | Script injection in output | Escape output | htmlspecialchars | |
| CSRF | Forged request from other site | Token in form | Prevention | |
| password_hash() | Hash password (bcrypt) | Never store plain password | Verification | |

### WordPress Core

| Concept | Definition | Simple Explanation | What Interviewers Test | Common Follow-up |
|---------|------------|--------------------|-------------------------|------------------|
| Core / Themes / Plugins | WP structure | Core = engine; themes = look; plugins = features | How WP works | |
| wp_posts, wp_users | DB tables | Content and users | Data model | |
| The Loop | WP_Query loop | Iterate posts | Template hierarchy | |
| Template hierarchy | page.php, single.php | Which file renders | Override in child theme | |
| Child theme | Extend parent theme | Safe updates | Best practice | |
| Hooks (Actions & Filters) | Extend without editing core | add_action, add_filter | VERY IMPORTANT | Priority, args |
| Shortcodes | [shortcode] → output | Dynamic content | When to use | |
| Custom Post Types (CPT) | Custom content type | Structured content | Register CPT | |
| Taxonomies | Categories/tags | Group content | Custom taxonomy | |
| REST API | Headless WP | JSON API | Modern apps | |
| WP_Query | Custom queries | Query posts | Performance (N+1) | |
| Transients | Temporary cache | DB-backed cache | When to use | |

### Security & Performance (WP)

| Concept | Definition | Simple Explanation | What Interviewers Test | Common Follow-up |
|---------|------------|--------------------|-------------------------|------------------|
| User roles | Capabilities | Admin, Editor, etc. | Access control | |
| Nonces | CSRF token | Verify request origin | Form security | |
| Object cache | Redis/Memcached | Speed up queries | Page cache vs object cache | |
| N+1 | Many queries in loop | Avoid with batch query | WP_Query optimization | |

---

## 3. How It Works (Internals)

**Request flow (WordPress)**  
Request → index.php → Load WP core → Parse query (URL → post/page) → Load theme → Template (e.g. single.php) → The Loop (WP_Query) → Output. Hooks (actions/filters) run at various points.

**Hooks**  
Action: do_action("name") – run callbacks added with add_action("name", callback). Filter: apply_filters("name", value) – callbacks modify value. Priority and number of args matter.

**Autoload (Composer)**  
PSR-4: namespace maps to path. require once when class is first used. Composer generates autoloader from composer.json.

**Word diagram**  
```
Request → [WP Bootstrap] → [Query Parse] → [Theme] → [Template] → [Loop] → HTML
                ↓
         [Actions / Filters]
```

---

## 4. Real-World Use Case

**Content site with custom features**

- **Where PHP/WP fits**: Content (posts, pages, CPT); themes for layout; plugins for forms, SEO, cache; REST API for mobile or headless frontend; custom plugin for business logic.
- **Why this approach**: Fast to launch; many themes/plugins; REST API for modern frontend; hooks for customization without forking core.
- **Trade-offs**: PHP and WP can be less structured than a custom API; scale and performance need caching and optimization.

---

## 5. Code Example (Minimal but Powerful)

```php
// PHP – prepared statement (no SQL injection), password, PDO
$pdo = new PDO("mysql:host=localhost;dbname=app", "user", "pass");

// CORRECT: Prepared statement
$stmt = $pdo->prepare("SELECT id, name FROM users WHERE email = ?");
$stmt->execute([$email]);
$user = $stmt->fetch(PDO::FETCH_ASSOC);

// WRONG: Concatenation – SQL injection
// $pdo->query("SELECT * FROM users WHERE email = '$email'");

// Password – never store plain
$hash = password_hash($password, PASSWORD_DEFAULT);
if (password_verify($input, $hash)) {
    // OK
}
```

```php
// WordPress – hook (filter), safe output
// In theme or plugin
add_filter("the_title", function ($title) {
    return $title . " – My Site";
}, 10, 1);

// Escape output – prevent XSS
echo esc_html(get_the_title());
echo esc_url($url);
// Wrong: echo get_the_title(); with user content → XSS risk
```

---

## 6. Best Practices (Interview-Grade)

| Do | Don't |
|----|--------|
| Use prepared statements (PDO/MySQLi) | Don't concatenate user input into SQL |
| Escape output (esc_html, esc_url) | Don't echo raw user content (XSS) |
| Use nonces for forms | Don't skip CSRF protection |
| Use password_hash/verify | Don't store or compare plain passwords |
| Use Composer and PSR-4 | Don't require files manually for classes |
| Use hooks in WP (don't edit core) | Don't modify core files |

**Security**: Input validation; output escaping; least privilege (DB user, file permissions); keep WP and plugins updated.  
**Performance**: Object cache (Redis); page cache; avoid N+1 (batch WP_Query); transients for expensive data.

---

## 7. Common Mistakes (Interview Red Flags)

| Mistake | Why wrong | Correct mental model |
|---------|-----------|------------------------|
| "We validate input so SQL injection is fine" | One bug = full compromise | Always use prepared statements |
| "Echo is fine for titles" | User-editable content can contain script | Always escape (esc_html, etc.) |
| "Edit core to add feature" | Updates overwrite core | Use theme/plugin and hooks |
| "Actions and filters are the same" | Action = do something; Filter = change value | Use add_action vs add_filter |
| "No need for nonces for logged-in users" | CSRF works against logged-in users | Use wp_nonce_field and verify |

---

## 8. Interview Questions & Answers

### A. Basic (Warm-up)

**Q: How do you prevent SQL injection in PHP?**  
**Short answer**: Use prepared statements (PDO or MySQLi). Bind parameters; never concatenate user input into SQL.  
**Follow-up**: "Is input validation enough?"  
**Ideal answer**: No. Validation is for business rules. One bypass or mistake and you get injection. Prepared statements are the fix.

**Q: What are WordPress hooks?**  
**Short answer**: Actions (do_action) run code at a point; filters (apply_filters) modify a value. Use add_action/add_filter to hook in without editing core.  
**Follow-up**: "When use action vs filter?"  
**Ideal answer**: Action when you want to do something (e.g. send email, log). Filter when you want to change a value (e.g. post title, content).

### B. Intermediate (Most common)

**Q: How do you prevent XSS in WordPress?**  
**Short answer**: Escape all output that can contain user or external data. Use esc_html(), esc_url(), esc_attr() depending on context. Never echo raw content in HTML.  
**Follow-up**: "What about content from the editor?"  
**Ideal answer**: Editor content is often allowed HTML (e.g. wp_kses_post). For full HTML from trusted source, output in a safe context. For untrusted, strip or escape.

**Q: What is N+1 in WordPress and how do you avoid it?**  
**Short answer**: In a loop, doing a query per item (e.g. get_post_meta per post) causes N+1 queries. Avoid: batch query (e.g. get all meta for post IDs), or use WP_Query with proper meta_query/relation so one query fetches what you need.

### C. Advanced / Tricky (Senior-level)

**Q: How would you design a custom plugin for a client?**  
**Ideal answer**: Single purpose; use hooks (actions/filters) only; namespace and PSR-4 autoload; config via options or customizer; escape output; use nonces for forms; use WP APIs (REST, cron, transients). Don't touch core. Document and version.

**Q: How do you secure WordPress in production?**  
**Ideal answer**: HTTPS; strong DB credentials; limit login attempts; nonce on forms; escape output; keep WP/plugins/themes updated; restrict file permissions; disable file edit in wp-config; use object cache and hardening (e.g. security plugin). Regular backups.

---

## 9. Quick Revision

- **SQL injection**: Use **prepared statements**; never concatenate input.
- **XSS**: **Escape output** (esc_html, esc_url).
- **CSRF**: **Nonces** on forms; verify on submit.
- **Passwords**: **password_hash** / **password_verify**.
- **WordPress**: **Hooks** (actions/filters); don't edit core.
- **Performance**: **Object cache**, avoid **N+1**, **transients**.

---

## 10. References

- [PHP Manual](https://www.php.net/manual/)
- [WordPress Developer Documentation](https://developer.wordpress.org/)
- [WordPress REST API Handbook](https://developer.wordpress.org/rest-api/)
