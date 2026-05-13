---
title: "Shadcn Registry: Reusable Components and Config Without Template Pain"
date: 2026-02-26 10:00:00
tags: [shadcn, registry, frontend, ai]
---

## 1. The problem: component libraries and project templates are hard to maintain

Software teams often build their own component library or project template for good reasons: design consistency across products, reuse instead of rebuilding the same patterns, and faster project setup with shared tooling and conventions. A custom component library keeps buttons, forms, and layouts consistent; a custom project template gives every new app the same lint rules, CI, and folder structure from day one. The intent is sound—centralize what's shared so teams can move faster and stay aligned.

The trouble is that maintaining these shared assets is hard. The pain shows up in two main ways.

**Component library as an npm package.** On a frontend team I worked on, we used to maintain our own component library as an npm package. To make a new component available, we had to make the UI code changes, publish to npm, and then upgrade the package version in every project that would use the new component. Sometimes we only found problems when we started using the new component in the actual project—and then we had to fix and go through the entire process again. One might argue that a monorepo could alleviate this (shared packages without the publish-and-upgrade cycle). The projects we built, however, had to stay isolated from one another (we operated as a venture studio—ownership, governance, and eventual spin-out all benefit from that separation). A monorepo is not a good fit for that model.

**Project template (golden repo).** We also found that managing our own project template was no easier. A "golden" repo—one big copy that every new project starts from—gets outdated quickly and is less flexible than we'd like. The frontend ecosystem moves fast: new frameworks and libraries show up all the time, so a template or tech stack that worked well a year ago—or even a few months ago—can feel outdated today. And once you've copied the template, existing projects don't get updates unless you manually backport changes. The template is tightly coupled to a fixed stack: a specific React version, bundler, folder structure, and tooling. Over time, projects drift. You end up with many slightly different copies and no single source of truth.

We also want to reuse more than just UI: GitHub Actions workflows, ESLint or Biome configs, and agent skills (e.g. for a CLI or tool). A single monolithic template forces all of that into one rigid package. What we need is a way to pull in *only* what we need, in small pieces, without locking the whole project to one stack.

## 2. Shadcn registry: piece-by-piece and decoupled

{% asset_img template-vs-reg.png Template vs registry: copy everything at once vs add only what you need %}

The [shadcn registry](https://ui.shadcn.com/docs/registry) takes a different approach. Instead of one template, you have a **catalog of items**—each a component or an arbitrary set of files—installable one at a time:

```bash
npx shadcn add <url>/r/<item>.json
```

There is no single "template." You add only what you need: one workflow, one ESLint config, one component. That keeps things less coupled to your tech stack. The schema is documented ([registry.json](https://ui.shadcn.com/docs/registry/registry-json), [registry-item.json](https://ui.shadcn.com/docs/registry/registry-item-json)), so you can build your own registry that follows the same format.

It's not just copy-paste. When you install a component from the registry, the CLI figures out which other components it depends on (and their indirect dependencies), installs all of them, and adds the required npm packages to your `package.json`. You get the right files and the right dependencies in one go—no manual wiring.

**Same mechanism for UI and non-UI.** The important part: the *same* registry format and CLI support both **`registry:component`** (UI components) and **`registry:file`** (workflows, ESLint, agent skills, config files). One pattern for many kinds of reuse—not just for components. So you can serve a React button, a GitHub Actions workflow, and an agent skill from the same registry, all via `npx shadcn add`.

**Ecosystem, not lock-in.** The shadcn CLI works with any registry URL. You can use the default shadcn registry for UI and your own registry for workflows, configs, and skills. Mix and match. That keeps you less coupled to any one vendor or repo.

## 3. You own the source (unlike npm)

With npm, you get a packaged dependency. To customize it, you often fork the package or wrap it—and then you own the fork. There is another pain with npm-based libraries: there are great open-source component libraries (Material UI, Mantine, and so on), but in practice you rarely use more than one in the same app. Each library brings its own design system, styling approach, and theming—mixing two full libraries usually means clashing styles, a bigger bundle, and an inconsistent UI. Yet you often want the best of each: one library's data grid, another's form components. With a registry like shadcn, you copy only the components you need into your repo. You own them from day one, so you can mix and match from multiple sources and keep one consistent look. You can edit them directly, and the registry is the single place you (or your team) go to add or update those files. No black box, no fighting with node_modules. That makes customization and long-term maintenance straightforward.

## 4. AI makes maintaining your own registry feasible

Maintaining a registry is not trivial either: copying files, wiring the JSON, writing docs, and running the build. That's why we leaned on AI to make the task easier—for example, an **agent skill** that encodes those steps. When you say "add this as a registry item," the AI does the mechanical work from that short instruction.

In practice, we ran an internal registry that served GitHub Actions workflows, ESLint/Biome configs, agent skills (e.g. for a CLI or tool), and UI components. Items were installed via `npx shadcn add <registry-url>/r/<item>.json`. An "add registry item" agent skill guided the AI to: copy source files into the registry's file tree, register the item in the root `registry.json`, run the registry build, and add a doc page. The doc page was part of a small documentation site built with Astro Starlight. That site did double duty: it hosted the docs (each item gets a page with its install command and description, so teammates can discover and use it) and served the registry itself—the JSON that the CLI fetches when you run `npx shadcn add`. So a human could say "add these files as a registry component" and the model would follow the checklist. That lowered the bar, so team members were more likely to maintain the registry and actually use it.

---

If you've been struggling with template drift or want to reuse workflows and configs without npm's lock-in, the registry pattern is worth a look. Same format, same CLI—for components and for everything else.
