---
layout: page
title: Frequently Asked Questions (FAQ)
navigation_source: docs_nav
---

### All my projects in one big repo?  Is that a good idea?
_Answered in [this article]({% link pages/intro/why_mono.md %})._

### Where do I send bug reports or feature requests?

Open a [GitHub issue](https://github.com/Microsoft/web-build-tools/issues) for the **web-build-tools** project.  Include "Rush" in your issue title.

### With many projects in one repo, will "npm install" take too long?

You might be thinking: "Hmmm.. if my current install takes 3 minutes, and you want me to put 20 projects in one repo, won't that multiply my NPM install time to 60 minutes!?"  Nope.  Rush centralizes your dependencies in a "common" folder and runs "npm install" exactly once, with essentially the same install time as your original monolithic application.

### Will Rush make my tooling nonstandard?

Nope!  Rush works within the existing systems and standards.  It just does things better and faster.
- Each project folder remains self-contained (no blurring of package boundaries)
- It's still possible to build any project without Rush; just do `npm install` and `gulp` as usual
- A project can be moved to a separate repo at any time, without any code changes; no commitment!

### Have you heard of Yarn or pnpm?

Yes, we're very interested in these tools!  They both promise to make `npm install` go a lot faster.  **Yarn** is a faster implementation of NPM's wacky node_modules layout, whereas **[pnpm](https://github.com/pnpm/pnpm)** replaces it with a different layout that greatly simplifies the install algorithm.  Since Rush runs `npm install` behind the scenes, and both of these tools are drop-in replacements for`npm install`, it would be pretty straightforward for Rush to support them.  Unfortunately both tools seem to be not quite production ready -- last we checked, neither is able to successfully install our production repositories.  :-(  As soon as that changes, we'll certainly jump on board.

### After installing Rush, why am I stilling see the old version?

This problem isn't specific to Rush, but we hear about it a lot because Rush is one of the first tools people need to invoke when starting work in a repo.  The symptoms look like this:

```
$ npm install -g @microsoft/rush
C:\Program Files\nodejs\rush -> C:\Program Files\nodejs\node_modules\@microsoft\
rush\bin\rush
C:\Program Files\nodejs
`-- @microsoft/rush@3.0.1

$ rush
Rush Multi-Package Build Tool 2.5.0 - http://aka.ms/rush
```

NPM seems to say that it is installing version 3.0.1, but when we execute the command, it shows Rush version 2.5.0.  What's going on here?!

The problem is that when you type commands like "gulp" or "rush", they are found in your system PATH, which can be pointing to folders from previous installs of NodeJS or NPM.

The fix:
1. Run `npm ls -g --depth 0` to figure out where your NPM packages are being installed.
2. Run the `set` command, and examine your PATH environment variable.
3. Make sure that no other NPM or NodeJS folders appear in your PATH before the folder from #1
4. Delete any obsolete folders from your PATH, e.g. from an old install of NPM, NodeJS, nodist, nvm-windows, etc.
5. If you previously used one of these alternative engines, most likely you have a bunch of deadwood NPM packages left behind on your disk somewhere.  It's a good idea to track them down and delete them.

Some places to look:
```
C:\Program Files\nodejs
C:\Program Files (x86)\nodist
%APPDATA%\npm
%APPDATA%\nvm
```

### The "npm install" step is reporting network errors -- what to do?

If you install packages from a custom NPM registry (e.g. a private server for your company or a caching proxy), then your project maintainer will instruct you to add special configuration settings in your .npmrc file.  If these settings are incorrect, "npm install" may report confusing errors that seem to indicate a network failure.  It's important to understand that the NPM tool looks for ".npmrc" files multiple locations (and ignores other locations).

Without Rush, NPM looks for "**.npmrc**" in these two places, *and merges their contents*:
- in the same folder as your package.json (useful for storing project-specific settings in Git)
- in your user home directory (your authentication token goes here)

When Rush invokes "npm install", it looks for "**.npmrc**" in these two places:
- "**./common/config/rush/.npmrc**" (which gets copied to "**./common/temp/.npmrc**" during install)
- in your user home directory
