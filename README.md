# ai:chat — An XPath 3.1 Extension Function for AI Chat Completion

This repository holds the specification of **`ai:chat`**, a proposed XPath 3.1
extension function for invoking AI language models from within XML processing
pipelines.

📖 **[Read the rendered specification](https://raw.githack.com/oxygenxml-incubator/ai-chat-spec/main/ai-chat-module.html)**

It is written as an **EXPath Candidate Module** and is designed to be:

- **Provider-independent** — the function signature exposes no provider-specific
  parameters. Backend selection, authentication, and model configuration are
  environment concerns, not author code.
- **Host-language agnostic** — usable from XSLT 3.0, XQuery 3.1, Schematron, SQF,
  XProc 3.0, and any other XPath 3.1 host language.
- **Composable** — the `?messages` array returned by a call can be passed
  straight into the next call, so multi-turn conversations need no external state.

## The function

```
ai:chat($messages as item()*, $opts as map(*)?) as map(*)
```

`$messages` is a sequence of message items (a plain `xs:string` is treated as a
`user` message; `map(*)` items carry an explicit `role`/`content`; `array(*)`
items are auto-expanded). `$opts` is an optional map of call-level settings
(`model`, `temperature`, `max-tokens`, `reasoning-effort`, `stop`, `cache`,
`on-error`, `tools`, …).

The result map carries `response` (the assistant reply as a string) and
`messages` (the full updated conversation history), plus optional `error` and
`warnings` keys.

### Examples

Single-turn, plain-string result:

```xquery
ai:chat('What is a DITA shortdesc?')?response
```

Multi-turn (the `?messages` array is auto-expanded into the next call):

```xquery
let $r1 := ai:chat('What is a shortdesc?')
let $r2 := ai:chat(($r1?messages, 'Give me an example.'))
return $r2?response
```

Inside a Schematron rule, for a semantic check:

```xml
<sch:assert test="
  ai:chat('Does this short description accurately summarise the following topic? '
          || .. || ' Shortdesc: ' || .
          || ' Answer yes or no.')?response = 'yes'">
  Short description may not match the topic.
</sch:assert>
```

The module also supports **tool / function calling**: when `$opts?tools`
supplies a list of tool definitions, a conforming implementation runs the
submit → tool-call → tool-result loop transparently and returns a single map,
preserving the full trace in `?messages`.

## Namespaces

| Prefix | Namespace URI |
| ------ | ------------- |
| `ai`   | `http://expath.org/ns/ai` |
| `err`  | `http://expath.org/ns/error` |

## Contents

- **[Rendered specification (HTML)](https://raw.githack.com/oxygenxml-incubator/ai-chat-spec/main/ai-chat-module.html)**
  — the readable version of the spec.
- [`ai-chat-module.xml`](ai-chat-module.xml) — the specification source, in the
  EXPath/W3C `<spec>` XML format.
- [`ai-chat-module.html`](ai-chat-module.html) — the HTML generated from the
  source (viewed through [raw.githack.com](https://raw.githack.com) so it
  renders instead of showing markup).

## Status

This is an **early-stage community proposal**. The function signatures and
behaviour reflect a working implementation in the Oxygen AI Positron add-on.
Feedback and alternative proposals are welcome — please open an issue.

## Author

George Bina — Syncro Soft / Oxygen XML Editor.

## License

This specification is licensed under the
[Creative Commons Attribution 4.0 International (CC BY 4.0)](LICENSE) license.
