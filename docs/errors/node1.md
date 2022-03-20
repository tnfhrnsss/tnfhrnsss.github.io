---
layout: post
title: Node Sass does not yet support your current environment
date: 2022-03-21 00:38:45
last_modified_at : 2022-03-21 00:38:45
parent: Errors
has_children: false
nav_exclude: true
tags: [node, saas]
---

# Node Sass does not yet support your current environment: Windows 64-bit with Unsupported runtime

에러 메시지:

```
Failed to compile.

./src/components/messages/common/Modal/Modal.scss (./node_modules/css-loader/dist/cjs.js??ref--6-oneOf-5-1!./node_modules/postcss-loader/src??postcss!./node_modules/resolve-url-loader??ref--6-oneOf-5-3!./node_modules/sass-loader/dist/cjs.js??ref--6-oneOf-5-4!./src/components/messages/common/Modal/Modal.scss)
Error: Node Sass does not yet support your current environment: Windows 64-bit with Unsupported runtime (83)
```

해결 : 

$ npm uninstall node-saas && npm install npde-saas

[Error: Node Sass does not yet support your current environment: Windows 64-bit with false](https://stackoverflow.com/a/41082773/14257397)