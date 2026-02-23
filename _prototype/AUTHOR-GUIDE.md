# Guides Navigation: Author Guide

This document is for the content author who needs to update the guides landing page, add or move guides, or customize navigation titles.

## Quick reference

All changes go in one file: `_data/domains.yaml`.

| Task | Section |
|------|---------|
| Add a new guide | [Add a guide to a domain](#add-a-guide-to-a-domain) |
| Add a guide to a subcategory | [Add a guide to a subcategory](#add-a-guide-to-a-subcategory) |
| Move a guide to a different domain | [Move a guide between domains](#move-a-guide-between-domains) |
| Reorder guides in the sidebar | [Reorder guides within a domain](#reorder-guides-within-a-domain) |
| Create a new subcategory | [Create a new subcategory](#create-a-new-subcategory) |
| Write or edit a sidebar title | [Set or change a nav-title](#set-or-change-a-nav-title) |
| Add guides after a release | [Update guides after a Quarkus release](#update-guides-after-a-quarkus-release) |
| Check for YAML errors | [Verify your changes](#verify-your-changes) |

## Add a guide to a domain

1. Open `_data/domains.yaml`
2. Find the domain (search for `id: build-backend-apis`, etc.)
3. Add a new entry to the `guides:` list at the position you want it to appear:

```yaml
  - url: /guides/your-new-guide
    title: Your New Guide Title
    type: howto
    nav-title: Set up your new feature
```

Always include a `nav-title` for new entries — the original titles are usually too long for the sidebar.

4. [Verify your changes](#verify-your-changes)

### Fields

| Field | Required | Description |
|-------|----------|-------------|
| `url` | Yes | The guide's URL path (e.g., `/guides/rest`) |
| `title` | Yes | The original guide title (matches the `.adoc` page title) |
| `type` | Yes | Content type from the guide's `.adoc` source metadata. One of: `tutorial`, `howto`, `guide`, `reference`, `concepts`. Copy this from the existing guide entry or from `quarkus.yaml` |
| `nav-title` | No | Short action-oriented title shown in the sidebar. Falls back to `title` if omitted |

Find `url`, `title`, and `type` for any existing guide in `_data/versioned/latest/index/quarkus.yaml` — search for the guide's filename (e.g., `rest-client`).

## Add a guide to a subcategory

Four large domains use subcategories to group related guides: Build Backend APIs, Deploy to the Cloud, Access and Manage Data, and Secure Your Application. A subcategory entry in `domains.yaml` looks like this:

```yaml
  - subcategory: gRPC
    guides:
    - url: /guides/grpc
      title: gRPC
      type: guide
      nav-title: Use gRPC
    - url: /guides/grpc-getting-started
      title: Getting Started with gRPC
      type: guide
      nav-title: Get started with gRPC
```

To add a guide to an existing subcategory, find the `- subcategory: <name>` block and append a new entry to its `guides:` list. The fields are the same as for top-level guides (`url`, `title`, `type`, `nav-title`).

To move a guide **into** a subcategory, cut its entry from the top-level `guides:` list and paste it into the subcategory's `guides:` list. To move a guide **out** of a subcategory, do the reverse.

Guides that don't belong to any subcategory sit at the top level of the domain's `guides:` list, before or between subcategory blocks.

## Create a new subcategory

Add a `- subcategory:` entry to the domain's `guides:` list with a nested `guides:` array:

```yaml
  - subcategory: Your Subcategory Name
    guides:
    - url: /guides/some-guide
      title: Some Guide Title
      type: howto
      nav-title: Do something
```

Subcategory blocks and top-level guides can be interleaved in any order — the YAML order is the display order. Currently only the 4 largest domains use subcategories; smaller domains stay flat.

## Move a guide between domains

Cut the entire entry (all its fields) from one domain's `guides:` list and paste it into another.

## Reorder guides within a domain

The order in `domains.yaml` is the order in the sidebar and on the landing page. Move the entry block up or down within the `guides:` list.

In domains with subcategories, you can reorder guides within a subcategory the same way. You can also reorder subcategory blocks relative to each other and to top-level guides — the YAML order is the display order.

Within each group, entries follow this order: tutorials, concepts, how-to guides, then reference. Place new entries next to related guides in the same topic area.

## Create a new domain

Add a new block to the `domains:` list in `_data/domains.yaml`:

```yaml
- id: your-domain-id
  title: Your Domain Title
  job: >-
    When I need to [situation], I want to [motivation],
    so I can [expected outcome].
  order: 16
  guides:
  - url: /guides/first-guide
    title: First Guide Title
    type: guide
    nav-title: Do the first thing
```

The `order` field controls the position in the sidebar and landing page. Set it higher than existing domains to add at the end, or renumber to insert.

## Set or change a nav-title

Many guide titles are too long or too technical for sidebar navigation. A `nav-title` provides a short, action-oriented alternative shown in the sidebar while the original `title` still appears on the guide page itself and in the landing page content area.

For example:

| Original title | nav-title |
|---|---|
| Writing REST Services with Quarkus REST (formerly RESTEasy Reactive) | Write REST services |
| Configuring Logging | Configure logging |
| OpenID Connect (OIDC) Authorization Code Flow Mechanism for Protecting Web Applications | Protect a web app with OIDC |
| All Configuration Properties | Configuration properties |

### Writing good nav-titles

Use action-oriented titles that describe what the user will do or learn. Match the verb pattern to the content type:

| Content type | Verb pattern | Examples |
|---|---|---|
| Tutorial | Build, Create, Protect, Get started | "Create your first application" |
| How-to | Configure, Enable, Use, Set up, Deploy | "Configure OIDC multitenancy" |
| Guide | Use, Write, Deploy, Connect, Schedule | "Use the REST client" |
| Concepts | Understand, Learn, Explore | "Understand security architecture" |
| Reference | Descriptive, no verb | "OIDC configuration properties" |

### Title guidelines

- Use imperative verbs, not gerunds: "Configure logging" not "Configuring logging"
- Focus on what the user does, not the feature name: "Protect a web app with OIDC" not "OpenID Connect authorization code flow mechanism"
- Keep titles concise: 3-7 words
- Strip redundant "Quarkus" prefix (the user is already in Quarkus docs)
- Strip "Reference Guide" suffix for references: "Stork reference" not "Stork Reference Guide"

### To add or change a nav-title

Edit the guide entry in `_data/domains.yaml`:

```yaml
  - url: /guides/rest
    title: Writing REST Services with Quarkus REST (formerly RESTEasy Reactive)
    type: guide
    nav-title: Write REST services    # <-- add or edit this line
```

If you omit `nav-title`, the sidebar shows the full `title`.

## Update guides after a Quarkus release

Add or remove entries in `domains.yaml` directly, following the format in [Add a guide to a domain](#add-a-guide-to-a-domain). For bulk rebuilds after major changes, see the [Maintainer Guide](MAINTAINER.md#maintaining-domainsyaml).

## Verify your changes

After editing `_data/domains.yaml`, check for YAML syntax errors:

```bash
python3 -c "import yaml; yaml.safe_load(open('_data/domains.yaml'))"
```

No output means the syntax is valid. To preview the rendered site, see the [Maintainer Guide](MAINTAINER.md#build-workflow) for local build instructions.

