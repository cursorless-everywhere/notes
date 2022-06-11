# cursorless-everywhere

Planning documents and meeting notes for the _Cursorless Everywhere_ project, which aims to bring [Cursorless](https://github.com/cursorless-dev/cursorless) to editors that are not VS Code, such as JetBrains editors, Emacs, etc.

## What are the different repositories?

- [cursorless](https://github.com/phillco/cursorless): this is our 
  fork of the main [Cursorless extension](https://github.com/cursorless-dev/cursorless).
  Any changes that are necessary to support the "everywhere" nature of the project
  go here. We aim to minimize changes needed here to support the project and try
  to keep it up to date with the upstream.
- [cursorless-sidecar](https://github.com/phillco/cursorless-sidecar): this 
  extension is where 1) we turn VS Code into a "sidecar", which runs in the 
  background and is manipulated by the "superior" editor, and allows us to get access to the Cursorless 
  logic without having to rewrite all of the code to not depend on VS Code 2) 
  a place to make all VS Code changes (such as exposing new commands) that 
  can avoid going in [cursorless](https://github.com/phillco/cursorless), 
  since we want to minimize the number of forked code changes there.
- [emacs-cursorless](https://github.com/rntz/emacs-cursorless): **[@rntz](https://github.com/rntz/)**'s 
  plugin to build an implementation for Emacs.
- _(TODO: publish!)_ **talon-intellij**: **[@phillco](https://github.com/phillco)**'s ground-up new version of the JetBrains 
  plugin. In addition to supporting Cursorless, it also proactively exposes 
  the state of every JetBrains instances using statefiles (namespaced by PID), 
  so Talon can watch them/query them more efficiently.
