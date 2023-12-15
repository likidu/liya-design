---
author: Liya Du
pubDatetime: 2023-12-14T21:38:00Z
title: Enable global errors and warnings in Visual Studio Code for your entire Typescript project
postSlug: containerized-wordpress-aws-tidb-serverless
featured: false
draft: false
tags:
  - devx
  - vscode
  - typescript
  - gist
ogImage: "/assets/20231214/og-global-problem-panel.png.png"
description: A quick gist to enable global errors and warnings in Visual Studio Code problem panel for your entire Typescript project.
---

As a Typescript developer and every day Visual Studio Code user, one of your best friends is the Problems panel. It shows you all the errors and warnings for the files you are opening. But what you really want is a "global view" of all the errors and warnings in your project. So you can fix them all at once.

Since in VSCode **v1.85**, now there is an experimental option to show all the errors and warnings in your project. You can enable it by adding the following line to your `settings.json` file:

```json
'typescript.tsserver.experimental.enableProjectDiagnostics': true
```

And it might work by immediately but there is a temporary fix for it:

1. Open an arbitrary Typescript file in your project.
1. Restart the TS Server by using `>TypeScript: Restart TS Server` from the command palette.

Wola! Now you can see all the errors and warnings in your project in the Problems panel not matter the file is opened or not.

<Image src="/assets/20231214/og-global-problem-panel.png" format="png" alt="Global errors and warnings problem panel" />

Really wish this feature will be stable soon. :crossed_fingers:
