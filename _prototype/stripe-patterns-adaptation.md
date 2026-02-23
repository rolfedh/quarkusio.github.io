# Prompt: Enhance the Quarkus Guides Landing Page with Stripe-Inspired Patterns

You are enhancing the Quarkus guides landing page at `/home/rdlugyhe/quarkus/quarkusio.github.io/`. The Phase 1 JTBD navigation redesign (15-domain taxonomy, persistent sidebar, prev/next navigation) is already deployed. You are adding three presentation-layer enhancements inspired by [docs.stripe.com/payments](https://docs.stripe.com/payments):

1. A **"Recommended first steps" hero section** above the domain list
2. **Concise use-case descriptions** replacing the JTBD job statements per domain
3. **Short nav-titles** in the landing page guide list instead of full titles

---

## Scope

**In scope (implement these):**
- Enhancement 1: "Recommended first steps" hero card row with 3 featured guides
- Enhancement 2: Rephrase domain job statements as concise, Stripe-style use-case descriptions; add `JOB_OVERRIDES` dict to `add-nav-titles.py` so they survive regeneration
- Enhancement 3: Show `nav-title` (with fallback to `title`) in the landing page guide list, matching the sidebar pattern

**Out of scope (do not implement):**
- CSS restyle of `.domain-job` (the existing italic styling is appropriate for the shorter text)
- Guide summary tooltips or visible summary text (deferred until editorial summaries are written)
- Complexity tiers within large domains
- Persona/audience indicator chips
- Changes to `search.quarkus.io` web components (Phase 2)
- Changes to the per-guide sidebar (`_includes/guide-sidebar.html`) or per-guide layout (`_layouts/guides.html`)
- Changes to `_layouts/documentation.html` routing logic

---

## Implementation order

These enhancements are independent and can be implemented in any order. All edits to `_data/domains.yaml` are direct — no scripts need to be re-run during implementation. The `JOB_OVERRIDES` and `FEATURED` dicts added to `add-nav-titles.py` exist solely to survive *future* regenerations.

Suggested order for a clean editing flow:

1. **Enhancement 3** (one-line Liquid change — fast win, immediately visible)
2. **Enhancement 2** (edit `job:` fields in `domains.yaml`, add `JOB_OVERRIDES` to `add-nav-titles.py`)
3. **Enhancement 1** (add `featured` fields to `domains.yaml`, add `FEATURED` to `add-nav-titles.py`, add hero section Liquid and SCSS)

---

## Design rationale

Stripe's payments landing page uses three patterns that the Phase 1 landing page lacks:

1. **"Most popular" highlighting**: Stripe surfaces 3-4 primary entry points above all other content, reducing cognitive load for newcomers. The Quarkus landing page currently renders all 15 domains with equal visual weight.

2. **Concise use-case framing**: Each Stripe section opens with a short, outcome-focused phrase (e.g., "Use Checkout to set up a Stripe-hosted page"). The Quarkus landing page has JTBD job statements written in internal "When I... I want... so I can..." format — useful for team alignment but not for end users scanning the page.

3. **Scannable link text**: Stripe uses short action-oriented labels. The Quarkus landing page shows full guide titles (e.g., "Writing REST Services with Quarkus REST (formerly RESTEasy Reactive)") while the sidebar already uses compact nav-titles (e.g., "Write REST services"). The main guide list should match the sidebar.

**What NOT to adopt from Stripe:**
- No-code paths (all Quarkus users write code)
- Product marketing in docs (Quarkus guides are purely navigational)
- Product silos (Quarkus extensions are composable, not siloed)
- Hardcoded per-feature styling (use CSS custom properties for dark mode)

---

## Enhancement 1: "Recommended first steps" hero section

Add a visually distinct card row above the domain sections showing the 3 most important starting-point guides.

```
+--------------------------------------------------------------+
|  RECOMMENDED FIRST STEPS                                     |
|  +----------------+  +----------------+  +----------------+  |
|  | Create your    |  | Write REST     |  | Deploy to      |  |
|  | first app      |  | services       |  | Kubernetes     |  |
|  |                |  |                |  |                |  |
|  | Build a hello- |  | Expose HTTP    |  | Package and    |  |
|  | world app in   |  | endpoints with |  | deploy your    |  |
|  | 10 minutes     |  | Quarkus REST   |  | app to K8s     |  |
|  |                |  |                |  |                |  |
|  | [Tutorial]     |  | [Guide]        |  | [Guide]        |  |
|  +----------------+  +----------------+  +----------------+  |
+--------------------------------------------------------------+
```

**Note:** The heading is "Recommended first steps", not "Start Here." The first domain section immediately below is "Get Started" — using "Start Here" would create a near-synonym collision.

### 1a. Add fields to `_data/domains.yaml`

Add `featured: true` and `featured-summary` to exactly these 3 guide entries (find them in their respective domains and add the fields):

```yaml
# In "Get Started" domain:
- url: /guides/getting-started
  title: Creating Your First Application
  type: tutorial
  nav-title: Create your first application
  featured: true
  featured-summary: "Build a hello-world app in 10 minutes"

# In "Build Backend APIs" domain:
- url: /guides/rest
  title: "Writing REST Services with Quarkus REST (formerly RESTEasy Reactive)"
  type: guide
  nav-title: Write REST services
  featured: true
  featured-summary: "Expose HTTP endpoints with JSON serialization"

# In "Deploy to the Cloud" domain:
- url: /guides/deploying-to-kubernetes
  title: Kubernetes extension
  type: guide
  nav-title: Deploy to Kubernetes
  featured: true
  featured-summary: "Package and deploy your app to Kubernetes"
```

### 1b. Add hero section to `_includes/index-docs-v2.html`

Insert the following block **after** the `<h1>` title (line 34) and **before** the `<qs-target>` tag (line 36). This placement keeps the hero section outside `<qs-target>` so search filtering does not hide it.

**CRITICAL — version-aware URLs.** Use the same URL construction pattern already used on line 66 of the file. Do NOT use bare `{{ guide.url }}` without the version prefix.

```liquid
{% comment %} Recommended first steps — 3 featured guides above the domain list {% endcomment %}
{% assign featured_guides = "" | split: "" %}
{% for domain in site.data.domains.domains %}
  {% for entry in domain.guides %}
    {% if entry.featured %}
      {% assign featured_guides = featured_guides | push: entry %}
    {% endif %}
  {% endfor %}
{% endfor %}

{% if featured_guides.size > 0 %}
<section class="start-here">
  <h2>Recommended first steps</h2>
  <div class="start-here__cards">
    {% for guide in featured_guides %}
    <a href="{% if docversion == 'latest' %}{{ site.baseurl }}{{ guide.url }}{% else %}{{ site.baseurl }}/version/{{ docversion }}{{ guide.url }}{% endif %}" class="start-here__card">
      <h3>{{ guide.nav-title | default: guide.title }}</h3>
      <p>{{ guide.featured-summary }}</p>
      <span class="type-badge">
        {% if guide.type == 'tutorial' %}Tutorial
        {% elsif guide.type == 'howto' %}How-to
        {% elsif guide.type == 'reference' %}Reference
        {% elsif guide.type == 'concepts' %}Concepts
        {% else %}Guide
        {% endif %}
      </span>
    </a>
    {% endfor %}
  </div>
</section>
{% endif %}
```

**Performance note:** The nested loop iterates all domains and all guides (~272 entries) to find 3 featured entries. This runs once per landing page render (1 page, not 268). Total iterations: ~272. This is fine — the O(n²) warning in `prototype-prompt.md` applies to inner loops that run on every guide page (268 pages × 272 entries = 73,000+ iterations).

### 1c. Make `featured` fields survive regeneration

The `generate-domains-yaml.py` script regenerates `domains.yaml` from scratch. Manually-added fields like `featured` and `featured-summary` are dropped on every regeneration, just as `nav-title` fields are. The `add-nav-titles.py` script solves this for `nav-title` by re-applying overrides after regeneration. Extend it to also apply `featured` fields.

In `_scripts/add-nav-titles.py`, add a `FEATURED` dict after the `OVERRIDES` dict (after line 310). This dict is keyed by URL (more stable than title for cross-domain lookups):

```python
# ── Featured guide overrides ─────────────────────────────────────────────
# key = guide URL, value = dict of fields to set.
# Applied after nav-title overrides so both survive regeneration.
FEATURED = {
    "/guides/getting-started": {
        "featured": True,
        "featured-summary": "Build a hello-world app in 10 minutes",
    },
    "/guides/rest": {
        "featured": True,
        "featured-summary": "Expose HTTP endpoints with JSON serialization",
    },
    "/guides/deploying-to-kubernetes": {
        "featured": True,
        "featured-summary": "Package and deploy your app to Kubernetes",
    },
}
```

Then, in the `add_nav_titles` function, add a second pass after the nav-title loop (after line 333, before the `if missing:` block):

```python
    # Apply featured fields by URL
    featured_count = 0
    for domain in data["domains"]:
        for guide in domain.get("guides", []):
            url = guide.get("url", "")
            if url in FEATURED:
                for key, value in FEATURED[url].items():
                    guide[key] = value
                featured_count += 1

    print(f"Featured fields applied: {featured_count}")
```

---

## Enhancement 2: Rephrase job statements as use-case descriptions

The current job statements use internal JTBD format ("When I'm evaluating Quarkus or starting a new project, I want to create a working application quickly, so I can understand the developer experience..."). Replace them with concise, outcome-focused descriptions inspired by [docs.stripe.com/payments/use-cases/get-started](https://docs.stripe.com/payments/use-cases/get-started).

**Tone:** Imperative verbs, 10-20 words, concrete outcomes. No "When I..." framing.

### 2a. Add `JOB_OVERRIDES` to `_scripts/add-nav-titles.py`

Add the following dict after the `FEATURED` dict (both are added after line 310 of the original file). This dict is keyed by domain `id`:

```python
# ── Job statement overrides ──────────────────────────────────────────────
# Replaces the JTBD "When I... I want... so I can..." statements with
# concise use-case descriptions (Stripe-style).
# key = domain id, value = new job text.
JOB_OVERRIDES = {
    "get-started": "Create a working application in minutes and explore the Quarkus developer experience.",
    "build-backend-apis": "Expose REST endpoints, consume external APIs, and build reliable service-to-service communication.",
    "build-web-uis": "Build full-stack web applications with server-side templates and real-time WebSocket connections.",
    "access-and-manage-data": "Connect to databases, query with ORM frameworks, manage schema migrations, and cache data.",
    "secure-your-application": "Add authentication, configure authorization, and protect endpoints with OIDC, JWT, or WebAuthn.",
    "send-and-receive-messages": "Produce and consume messages with Kafka, AMQP, Pulsar, or RabbitMQ to decouple services.",
    "deploy-to-the-cloud": "Build container images, compile native executables, and deploy to Kubernetes, OpenShift, or serverless platforms.",
    "observe-your-application": "Collect metrics, traces, and logs with OpenTelemetry and Micrometer to monitor production services.",
    "understand-the-runtime": "Learn how CDI, the reactive engine, virtual threads, and Dev Services work under the hood.",
    "automate-and-integrate": "Schedule tasks, send email, apply business rules, and integrate with external systems.",
    "use-build-tools": "Configure Maven or Gradle, measure test coverage, and optimize your build workflow.",
    "use-spring-apis": "Use familiar Spring DI, Web, Data, and Security annotations as a migration bridge from Spring Boot.",
    "write-extensions": "Build custom extensions with build-time optimizations and contribute reusable functionality to the ecosystem.",
    "contribute-to-quarkus-docs": "Write documentation that meets the project's content standards and quality bar.",
    "build-ai-applications": "Integrate large language models, RAG pipelines, and AI services into your Java application.",
}
```

### 2b. Add a job-override pass in `add_nav_titles()`

In the `add_nav_titles` function, add a pass after the featured-fields pass (and before the `if missing:` block) that overwrites domain job statements:

```python
    # Apply job statement overrides by domain id
    job_count = 0
    for domain in data["domains"]:
        domain_id = domain.get("id", "")
        if domain_id in JOB_OVERRIDES:
            domain["job"] = JOB_OVERRIDES[domain_id]
            job_count += 1

    print(f"Job overrides applied: {job_count}")
```

### 2c. Edit `job:` fields directly in `_data/domains.yaml`

For each of the 15 domains, find the `job:` field and replace the JTBD statement with the matching value from `JOB_OVERRIDES`. Do NOT re-run `generate-domains-yaml.py` — that would overwrite the edited values with the old JTBD text. The `JOB_OVERRIDES` dict in `add-nav-titles.py` ensures these values survive future regenerations.

---

## Enhancement 3: Show nav-titles on the landing page

The landing page sidebar (line 48 of `index-docs-v2.html`) already uses `{{ entry.nav-title | default: entry.title }}`. The main guide list (line 66) still uses `{{ entry.title }}`, showing long full titles like "Writing REST Services with Quarkus REST (formerly RESTEasy Reactive)". Make the guide list consistent with the sidebar.

**Find this** on line 66 of `_includes/index-docs-v2.html`:
```liquid
              <a href="{% if docversion == 'latest' %}{{ site.baseurl }}{{ entry.url }}{% else %}{{ site.baseurl }}/version/{{ docversion }}{{ entry.url }}{% endif %}">{{ entry.title }}</a>
```

**Replace with:**
```liquid
              <a href="{% if docversion == 'latest' %}{{ site.baseurl }}{{ entry.url }}{% else %}{{ site.baseurl }}/version/{{ docversion }}{{ entry.url }}{% endif %}">{{ entry.nav-title | default: entry.title }}</a>
```

The only change is `{{ entry.title }}` → `{{ entry.nav-title | default: entry.title }}`. Guides without a `nav-title` field fall back to the full title.

---

## SCSS additions

Add the following styles to `_sass/layouts/guide-sidebar.scss`, **after** the `.domain-section` closing brace (line 172):

```scss
/* "Recommended first steps" hero section — featured guide cards above the domain list.
 * Placed outside .domain-section to avoid inheriting domain list styles.
 *
 * DARK MODE: All colors use CSS custom properties from colormode.scss.
 * Never hardcode hex colors. */
.start-here {
  margin-bottom: 2.5rem;
  padding: 1.5rem;
  background: var(--sec-background-color);
  border-radius: 8px;
  border: 1px solid var(--card-outline, #e1e4e8);

  h2 {
    margin-top: 0;
    margin-bottom: 1rem;
  }

  &__cards {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
    gap: 1rem;

    @media screen and (max-width: 768px) {
      grid-template-columns: 1fr;
    }
  }

  &__card {
    display: block;
    padding: 1rem 1.25rem;
    border: 1px solid var(--card-outline, #e1e4e8);
    border-radius: 6px;
    background: var(--card-background-color, #fff);
    color: var(--main-text-color);
    text-decoration: none;
    transition: border-color 0.15s;

    &:hover {
      border-color: var(--link-color, #4695EB);
      text-decoration: none;
    }

    h3 {
      margin: 0 0 0.5rem;
      font-size: 1rem;
      color: var(--link-color, #4695EB);
    }

    p {
      margin: 0 0 0.75rem;
      font-size: 0.9rem;
      line-height: 1.4;
      color: var(--main-text-color);
      opacity: 0.8;
    }

    .type-badge {
      font-size: 0.75rem;
      opacity: 0.6;
      font-family: monospace;
      text-transform: uppercase;
    }
  }
}
```

**SPECIFICITY NOTE:** The `.start-here` block is a top-level class outside `.guide-sidebar` and `.domain-section`, so it avoids the four global CSS hazards documented in the `guide-sidebar.scss` header comment (lines 6-17).

---

## Current site architecture (reference)

Key files this prompt touches and their roles:

| File | Role | Action |
|------|------|--------|
| `_data/domains.yaml` | Domain taxonomy with guide entries (url, title, type, nav-title) | Edit — add `featured`, `featured-summary` fields to 3 guides |
| `_includes/index-docs-v2.html` | Domain-grouped landing page template. Renders guide lists inside `<qs-target>` for search compatibility | Edit — add hero section after `<h1>`, use `nav-title` in guide list (line 66) |
| `_sass/layouts/guide-sidebar.scss` | Sidebar, prev/next, domain section styles | Edit — add `.start-here` styles after `.domain-section` |
| `_scripts/add-nav-titles.py` | Applies `nav-title` overrides after regeneration (lines 25-310: `OVERRIDES` dict; lines 313-348: `add_nav_titles` function) | Edit — add `FEATURED` dict, `JOB_OVERRIDES` dict, and two new passes in `add_nav_titles` |

Files NOT touched (verify you don't modify these):

| File | Why not |
|------|---------|
| `_includes/guide-sidebar.html` | Per-guide sidebar — no changes in this prompt |
| `_layouts/guides.html` | Per-guide layout — no changes |
| `_layouts/documentation.html` | Landing page router — no changes |
| `_includes/index-doc-item.html` | `<qs-guide>` renderer — not used on the new landing page |
| `_sass/layouts/guides.scss` | `qs-guide` card styles — not used on the new landing page |
| `_scripts/generate-domains-yaml.py` | No changes — job overrides are applied in `add-nav-titles.py` after regeneration |

---

## Constraints

- **Pure Jekyll**: Liquid templates, SCSS, and Python scripts (dev tools). No new npm/gem dependencies.
- **No URL changes**: guide URLs stay the same.
- **Search compatibility**: The hero section is placed OUTSIDE `<qs-target>` (between `<h1>` and `<qs-target>`) so it is always visible regardless of search state. Guide entries inside `<qs-target>` remain plain `<li>` elements (not `<qs-guide>` — the Phase 1 landing page already moved away from `<qs-guide>` cards).
- **Dark mode**: Use CSS custom properties from `_sass/colormode.scss`. Never hardcode hex colors. The fallback values in `var(--prop, #fallback)` are for browsers that don't support custom properties, not for dark mode — dark mode overrides are handled by `html.dark` in `colormode.scss`.
- **CSS specificity**: New styles are scoped to `.start-here` (top-level). This does not trigger the four global CSS hazards (`nav { float }`, `nav ul li { float }`, `nav ul li a { font-size }`, `.guides ul li { margin-bottom }`) because it doesn't use `<nav>` elements or unscoped `.guides` children.
- **Incremental builds**: SCSS changes are picked up by incremental builds. Changes to `_includes/` require a full rebuild (stop and restart the container).

---

## Testing

A Jekyll dev server runs in a podman container named `quarkus-jekyll` on port 4000 with `--incremental` mode. It auto-rebuilds when files change.

To restart if needed:
```bash
cd /home/rdlugyhe/quarkus/quarkusio.github.io
podman stop quarkus-jekyll 2>/dev/null; podman rm quarkus-jekyll 2>/dev/null
podman run --rm --name quarkus-jekyll -d -p 4000:4000 -v .:/site:Z \
  bretfisher/jekyll-serve \
  bundle exec jekyll serve --force_polling --incremental -H 0.0.0.0 -P 4000 \
  --config _config.yml,_config_dev.yml
```

**Use `http://127.0.0.1:4000`** (IPv4), not `localhost` — the container binds to `0.0.0.0` and `localhost` may resolve to IPv6 `::1` which won't connect.

Check build status with `podman logs --tail 5 quarkus-jekyll`. Look for `done in X seconds`.

**After changing `_includes/index-docs-v2.html`**, stop and restart the container (incremental builds do not pick up `_includes/` changes). Wait for the full rebuild (~5 minutes). After changing only `_sass/` files, incremental rebuild suffices (~3 minutes).

### Verification steps

After every build completes, use Ctrl+Shift+R for a hard refresh to bypass cached CSS, then verify:

1. **Hero section renders**: Open `http://127.0.0.1:4000/guides/`. The "Recommended first steps" section appears above the first domain ("Get Started"). It shows 3 cards with titles, summaries, and type badges. The heading says "Recommended first steps" — not "Start Here."
2. **Hero cards link correctly**: Click each hero card. It navigates to the correct guide page. Back-navigate and confirm.
3. **Job statements rephrased**: Each domain section shows a concise use-case description (e.g., "Expose REST endpoints, consume external APIs, and build reliable service-to-service communication.") — not the old JTBD format ("When I need to expose HTTP endpoints..."). Verify all 15 domains have the new text.
4. **Nav-titles in guide list**: The main guide list shows short nav-titles (e.g., "Write REST services") instead of full titles (e.g., "Writing REST Services with Quarkus REST (formerly RESTEasy Reactive)"). Guides without a nav-title fall back to the full title.
5. **Search still works**: Type `kafka` in the search bar at the top. Confirm that only Kafka-related guides remain visible. Confirm the "Recommended first steps" hero section remains visible (it is outside `<qs-target>`). Clear the search and confirm all guides reappear.
6. **Version dropdown**: Switch to an older version (e.g., 3.15). Confirm the landing page renders without errors. The hero section and domain list should render from `domains.yaml` (version-independent). Guide links should include the `/version/3.15/` prefix.
7. **Dark mode**: Toggle dark mode. Confirm the hero cards use appropriate colors (no white-on-white or unreadable text).
8. **Mobile**: Resize the browser to <768px. Confirm the sidebar hides (`hide-mobile`). Confirm the hero cards stack into a single column (not 2+1 orphan). Confirm the 2-column guide list collapses to 1 column.
9. **No layout regressions**: Scroll through all 15 domain sections. Confirm no broken spacing, overlapping elements, or visual glitches compared to the pre-enhancement state.
10. **Regeneration survival**: Run `python3 _scripts/generate-domains-yaml.py && python3 _scripts/add-nav-titles.py` and verify that `_data/domains.yaml` has: (a) the 3 `featured: true` guides with `featured-summary` fields, (b) all 15 rephrased `job:` statements, and (c) all `nav-title` fields.
