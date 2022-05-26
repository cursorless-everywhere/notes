# Milestones

(What's next for the prototype?)

## M0.1:

### Goals:

- JetBrains will write out the contents of the active editor to a temporary
  file (or just a file that's namespaced under the JetBrains state folder) for
  VS Code to view.
    - VS Code will _not_ open the actual file, because we don't want to have to
      deal with its saving its changes to that file and then dealing with the
      JetBrains "file on disk is different from memory contents" dialog.
    - Instead, all cursorless edits need to be applied as if they were real user
      changes (they need to appear in the undo stack, etc)
    - The JetBrains user should not need to save their file for Cursorless to
      work.
- We will change the sidecar extension to load this temporary file rather than
  the real one.
    - Every time the JetBrains state changes we will clobber the temporary file
      and have the VS Code reload it.
    - Long term, this is not ideal because Cursorless depends on receiving diffs
      from the VS Code to manage its state.
        - It can handle the file completely reloading, but this could cause the
          hat map to change, which would prevent chaining multiple cursorless
          commands within a single utterance.
        - However, for this milestone chaining probably won't work in any capacity,
          so will come back to this later.
- JetBrains will record serial numbers/versions for every editor (or for the
  state as a whole -- pokey's suggestion).
    - We will attempt to tag the hats file with the serial number, so we can
      tell quickly if we are viewing a stale hats file, and ignore it
- We then need to identify when cursorless commands have run, and intelligently
  apply that diff back to JetBrains.
    - As above, it would be best if this was a precise diff, rather than "load
      this alternate version of the file" so that we don't have to worry about
      race conditions on JetBrains side (e.g., the user also making keyboard
      changes at the same time).
        - I imagine this would make the undo experience better as well.
        - This is therefore significantly more important than sending smart
          diffs to VS Code.
    - VS Code provides existing watching APIs that we could used to listen to
      changes, including those made by cursorless [1].
        - These are nice because they return precise diffs, so we wouldn't have
          to do that. We would need to filter out changes caused by us
          synchronizing with JetBrains, however.
        - One note: the Cursorless "execute command" promise guarantees that all
          VS Code watchers that have been registered on the document are
          notified _before it returns_, which is nice.
        - This means, if we are the ones executing the Cursorless commands, we
          could possibly listen for and collect all of the changes that were
          made during the course of that command, returning them as a structured
          object.
    - Alternatively, we could run stuff after the `cursorless_command`
      action finishes, and try to figure out what changed on the VS Code side.
        - Since VS Code is working in a temporary file, we could have it save
          its work and diff it against the original version.
        - We could have it serialize its new cursor locations and just apply
          them blindly to JetBrains. Again, risk of racing here.
        - N.B., I (Phil) do not care about modifying/forking the cursorless code
          as much as pokey does; pokey would prefer everything happen in the
          sidecar plugin and minimize changes to cursorless itself.

### More details

- VS Code plugin side:
    - The sidecar extension will publish a callback function which Cursorless
      can call into whenever its hats change (or a generic K/V API)
    - The sidecar extension will now be what writes the hats file out, not
      Cursorless.
    - Since the sidecar extension will be the thing that also received the
      synchronization request from JetBrains (by watching its state file), it
      can thus also include the JetBrains state version/serial to the hats file
        - This isn't 100% guaranteed to be race-proof if multiple JetBrains
          changes come in quickly, but hopefully it's close enough

### Problems:

1. Cursorless commands need to be blocking, and when you use VS Code, they are
   because of the command server (the action does not return until the command
   is fully applied.).
    - However, with the sidecar, the _transmission_ of that cursorless change
      back to JetBrains would be asynchronous.
    - This means that something like a cursorless command followed by "air"
      would have those two commands race.
        - So you won't really be able to chain cursorless commands alongside
          other Talon actions unless they do not logically affect the same part
          of the code.
        - This is fine for M0.1 because the goal is just to get something
          working, but will to revisit this for M0.2.

2. The VS Code command server requires a keystroke to be triggered for it to "
   pick up" the command and respond to it. This requires that VS Code be
   focused, which obviously wouldn't be the case if the user intends it to be a
   background sidecar.
    - On macOS we can work around this because there's an API to send keystrokes
      to non focused applications, but not on other platforms.
    - Solution: if you don't care about using VS Code outside of cursorless
      everywhere / are less paranoid about security, we could fork the command
      server to be an HTTP server which would not require this keystroke, just
      for this prototype.
        - Why wasn't it an HTTP server originally? Security concerns from a
          user. However it shouldn't be a huge deal -- the existing JetBrains
          plugin uses a HTTP server with a nonce file.
        - The other reason we have the keystroke is to provide a synchronization
          barrier if the user is using VS Code as their primary editor.
            - Many Talon commands insert by mimicking keystrokes, so the
              shortcut helps ensure that cursorless commands, which would
              otherwise run via the asynchronous command server, don't race with
              the keyboard.
            - But this isn't important for the Cursorless everywhere user, for
              whom VS Code only exists to run in the background and handle
              cursorless commands.
        - That should unblock @rntz for now.

### Other TODOs:

- [ ] Make the sidecar close other tabs when switching files, right now they
accumulate
- [ ] Change the viewport logic so that we center around the cursor, rather than
just scrolling to the first visible line in JetBrains -- otherwise hats won't
render if the VS Code window isn't tall enough

# Appendix

[1] mThe VS Code watcher API:

```
      // An Event which fires when the active editor has changed. Note that the event also fires when the active editor changes to undefined.
      vscode.window.onDidChangeActiveTextEditor(this.addDecorationsDebounced),
      // An Event which fires when the array of visible editors has changed.
      vscode.window.onDidChangeVisibleTextEditors(this.addDecorationsDebounced),
      // An event that is emitted when a text document is changed. This usually happens when the contents changes but also when other things like the dirty-state changes.
      vscode.workspace.onDidChangeTextDocument(this.addDecorationsDebounced),
      // An Event which fires when the selection in an editor has changed.
      vscode.window.onDidChangeTextEditorSelection(
        this.addDecorationsDebounced
      ),
      // An Event which fires when the visible ranges of an editor has changed.
      vscode.window.onDidChangeTextEditorVisibleRanges(
        this.addDecorationsDebounced
      )
```
