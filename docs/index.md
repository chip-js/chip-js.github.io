---
layout: page
menuTitle: Docs
title: Chip Documentation
---

## Getting Started



## Many Modules

Chip is made up of many smaller modules. These modules are created such that you could roll your own framework if you
wanted to. But the real benefit is that each module is focused on one specific job.

* **[chip-js](chip-js/)** A powerful JavaScript framework

* **[fragments-js](fragments-js/)** An ultra-fast templating and data-binding library for front-end JavaScript
  applications.

* **[observations-js](observations-js/)** Observes simplified JavaScript expressions and triggers callbacks when the
  returned value changes.

* **[expressions-js](expressions-js/)** Converts simplified JavaScript expressions into executable functions for
  JavaScript frameworks.

* **[differences-js](differences-js/)** Checks if there are any differences between two objects and provides the
  difference.

* **[built-ins](built-ins/)** Built in binders, formatters, and animations for fragments.js-based frameworks.

* **[routes-js](routes-js/)** A little standalone routing library, used in chip.js

* **[chip-utils](chip-utils/)** A few basic utilities for use within chip-supported libraries.

```
chip-js
  │
  ├──fragments-js
  │    │
  │    └──observations-js
  │         │
  │         ├──expressions-js
  │         │
  │         └──differences-js
  │
  ├──built-ins
  │
  └──routes-js
```