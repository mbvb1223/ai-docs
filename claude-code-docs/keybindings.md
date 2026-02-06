# Customize keyboard shortcuts

> Customize keyboard shortcuts in Claude Code with a keybindings configuration file.

Claude Code supports customizable keyboard shortcuts. Run `/keybindings` to create or open your configuration file at `~/.claude/keybindings.json`.

## Configuration file

The keybindings configuration file is an object with a `bindings` array. Each block specifies a context and a map of keystrokes to actions.

> **Note:** Changes to the keybindings file are automatically detected and applied without restarting Claude Code.

| Field      | Description                                        |
| :--------- | :------------------------------------------------- |
| `$schema`  | Optional JSON Schema URL for editor autocompletion |
| `$docs`    | Optional documentation URL                         |
| `bindings` | Array of binding blocks by context                 |

This example binds `Ctrl+E` to open an external editor in the chat context, and unbinds `Ctrl+U`:

```json
{
  "$schema": "https://www.schemastore.org/claude-code-keybindings.json",
  "$docs": "https://code.claude.com/docs/en/keybindings",
  "bindings": [
    {
      "context": "Chat",
      "bindings": {
        "ctrl+e": "chat:externalEditor",
        "ctrl+u": null
      }
    }
  ]
}
```

## Contexts

Each binding block specifies a **context** where the bindings apply:

| Context           | Description                                      |
| :---------------- | :----------------------------------------------- |
| `Global`          | Applies everywhere in the app                    |
| `Chat`            | Main chat input area                             |
| `Autocomplete`    | Autocomplete menu is open                        |
| `Settings`        | Settings menu (escape-only dismiss)              |
| `Confirmation`    | Permission and confirmation dialogs              |
| `Tabs`            | Tab navigation components                        |
| `Help`            | Help menu is visible                             |
| `Transcript`      | Transcript viewer                                |
| `HistorySearch`   | History search mode (Ctrl+R)                     |
| `Task`            | Background task is running                       |
| `ThemePicker`     | Theme picker dialog                              |
| `Attachments`     | Image/attachment bar navigation                  |
| `Footer`          | Footer indicator navigation (tasks, teams, diff) |
| `MessageSelector` | Rewind dialog message selection                  |
| `DiffDialog`      | Diff viewer navigation                           |
| `ModelPicker`     | Model picker effort level                        |
| `Select`          | Generic select/list components                   |
| `Plugin`          | Plugin dialog (browse, discover, manage)         |

## Available actions

Actions follow a `namespace:action` format, such as `chat:submit` to send a message or `app:toggleTodos` to show the task list.

### App actions

| Action                 | Default | Description                 |
| :--------------------- | :------ | :-------------------------- |
| `app:interrupt`        | Ctrl+C  | Cancel current operation    |
| `app:exit`             | Ctrl+D  | Exit Claude Code            |
| `app:toggleTodos`      | Ctrl+T  | Toggle task list visibility |
| `app:toggleTranscript` | Ctrl+O  | Toggle verbose transcript   |

### Chat actions

| Action                | Default        | Description              |
| :-------------------- | :------------- | :----------------------- |
| `chat:cancel`         | Escape         | Cancel current input     |
| `chat:cycleMode`      | Shift+Tab      | Cycle permission modes   |
| `chat:modelPicker`    | Cmd+P / Meta+P | Open model picker        |
| `chat:thinkingToggle` | Cmd+T / Meta+T | Toggle extended thinking |
| `chat:submit`         | Enter          | Submit message           |
| `chat:undo`           | Ctrl+_         | Undo last action         |
| `chat:externalEditor` | Ctrl+G         | Open in external editor  |
| `chat:stash`          | Ctrl+S         | Stash current prompt     |
| `chat:imagePaste`     | Ctrl+V         | Paste image              |

### History actions

| Action             | Default | Description           |
| :----------------- | :------ | :-------------------- |
| `history:search`   | Ctrl+R  | Open history search   |
| `history:previous` | Up      | Previous history item |
| `history:next`     | Down    | Next history item     |

### Autocomplete actions

| Action                  | Default | Description         |
| :---------------------- | :------ | :------------------ |
| `autocomplete:accept`   | Tab     | Accept suggestion   |
| `autocomplete:dismiss`  | Escape  | Dismiss menu        |
| `autocomplete:previous` | Up      | Previous suggestion |
| `autocomplete:next`     | Down    | Next suggestion     |

### Confirmation actions

| Action                      | Default   | Description                   |
| :-------------------------- | :-------- | :---------------------------- |
| `confirm:yes`               | Y, Enter  | Confirm action                |
| `confirm:no`                | N, Escape | Decline action                |
| `confirm:previous`          | Up        | Previous option               |
| `confirm:next`              | Down      | Next option                   |
| `confirm:nextField`         | Tab       | Next field                    |
| `confirm:cycleMode`         | Shift+Tab | Cycle permission modes        |
| `confirm:toggleExplanation` | Ctrl+E    | Toggle permission explanation |

## Keystroke syntax

### Modifiers

Use modifier keys with the `+` separator:

* `ctrl` or `control` - Control key
* `alt`, `opt`, or `option` - Alt/Option key
* `shift` - Shift key
* `meta`, `cmd`, or `command` - Meta/Command key

For example:

```
ctrl+k          Single key with modifier
shift+tab       Shift + Tab
meta+p          Command/Meta + P
ctrl+shift+c    Multiple modifiers
```

### Chords

Chords are sequences of keystrokes separated by spaces:

```
ctrl+k ctrl+s   Press Ctrl+K, release, then Ctrl+S
```

### Special keys

* `escape` or `esc` - Escape key
* `enter` or `return` - Enter key
* `tab` - Tab key
* `space` - Space bar
* `up`, `down`, `left`, `right` - Arrow keys
* `backspace`, `delete` - Delete keys

## Unbind default shortcuts

Set an action to `null` to unbind a default shortcut:

```json
{
  "bindings": [
    {
      "context": "Chat",
      "bindings": {
        "ctrl+s": null
      }
    }
  ]
}
```

## Reserved shortcuts

These shortcuts cannot be rebound:

| Shortcut | Reason                     |
| :------- | :------------------------- |
| Ctrl+C   | Hardcoded interrupt/cancel |
| Ctrl+D   | Hardcoded exit             |

## Terminal conflicts

Some shortcuts may conflict with terminal multiplexers:

| Shortcut | Conflict                          |
| :------- | :-------------------------------- |
| Ctrl+B   | tmux prefix (press twice to send) |
| Ctrl+A   | GNU screen prefix                 |
| Ctrl+Z   | Unix process suspend (SIGTSTP)    |

## Vim mode interaction

When vim mode is enabled (`/vim`), keybindings and vim mode operate independently:

* **Vim mode** handles input at the text input level (cursor movement, modes, motions)
* **Keybindings** handle actions at the component level (toggle todos, submit, etc.)
* The Escape key in vim mode switches INSERT to NORMAL mode; it does not trigger `chat:cancel`
* Most Ctrl+key shortcuts pass through vim mode to the keybinding system

## Validation

Claude Code validates your keybindings and shows warnings for:

* Parse errors (invalid JSON or structure)
* Invalid context names
* Reserved shortcut conflicts
* Terminal multiplexer conflicts
* Duplicate bindings in the same context

Run `/doctor` to see any keybinding warnings.
