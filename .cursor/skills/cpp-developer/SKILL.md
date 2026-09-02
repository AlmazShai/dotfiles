---
name: cpp-developer
description: C++23 development conventions - DRY/KISS/YAGNI/SOLID priority, terse comments, hot-path latency, mandatory clang-format, and the dev-container build. Use when writing, editing, or refactoring C++ sources, headers, or CMake files.
paths:
  - "**/*.cpp"
  - "**/*.cc"
  - "**/*.cxx"
  - "**/*.h"
  - "**/*.hpp"
  - "**/*.ipp"
  - "**/CMakeLists.txt"
  - "**/*.cmake"
---

# C++ development

## Language and design

- Target modern C++23. Prefer current facilities over legacy equivalents: `std::span`,
  `std::string_view`, `std::expected`, ranges, `constexpr`/`consteval`, `std::format`,
  designated initializers.
- Apply DRY, KISS, YAGNI and SOLID, in that priority order. When two conflict, the earlier one
  wins: remove the duplication, keep the result simple, add nothing not needed yet, and only
  then reach for SOLID structure.
- SOLID ranks last because its abstractions cost the most. Do not introduce an interface, base
  class, or injection point for a variation that does not exist today.
- Follow the conventions already present in the file being edited over any general preference
  stated here.

## Comments and documentation

Short by default; length must be earned.

- Do not write doc comments. No `///`, `/** */`, `@brief`, `@param`, `@return` or file/class/
  function header blocks on anything new or edited, unless the user explicitly asks for
  documentation. Leave doc comments that already exist alone.
- Rename before you annotate. Comment only what a better name cannot fix.
- Never restate what the code says, explain the change being made, or address the reviewer.
- One line is the norm. Prefer a clause to a sentence, a sentence to a paragraph.
- When documentation is explicitly requested, state intent plus only the parameters, returns, or
  preconditions a caller cannot infer from the signature. Do not fill in every `@param` for
  completeness.

## Error handling

- Throw with `TRD_TRACE_EXCEPTION(fmt, ...)`, or `TRD_TRACE_EXCEPTION_T(ExceptionType, fmt, ...)`
  when a specific exception type is needed (`core/exceptions/exceptions_trace.hpp`).
- Never use `TRD_EXCEPTION` or `TRD_EXCEPTION_T`: they throw without attaching a stacktrace, so the
  failure cannot be traced back to its origin. Replace them with the `TRD_TRACE_*` equivalents in
  any code being touched.

## Hot paths

`Apply` and `Process` functions are normally the latency-critical path. Inside them and anything
they call:

- No allocation, locking, I/O, or logging on the happy path. Reserve and reuse buffers owned by
  the object instead of building temporaries per call.
- Avoid `std::function`, virtual dispatch, and shared-ownership copies. Pass `std::span`,
  `std::string_view`, or const references; take `shared_ptr` by const reference when unavoidable.
- Hoist loop-invariant work out of loops, and anything that can be computed at construction or
  configuration time out of the call entirely.
- Keep the common branch first and push error and cold handling into separate non-inlined
  functions. Use `[[likely]]`/`[[unlikely]]` only where the skew is real.
- Prefer contiguous, cache-friendly layouts over node-based containers.

When a latency-motivated choice makes the code harder to read, that is the one case where a short
comment stating the constraint is warranted.

## Formatting is mandatory

After editing any C++ file, format it with the project's own `.clang-format`. Use the container's
clang-format, since its version is the one CI formats with:

```bash
f=/abs/path/to/File.cpp
~/.config/nvim-ide/bin/clang-format-devc "$f" < "$f" > "$f.fmt" && mv "$f.fmt" "$f"
```

If the container is not running, fall back to the host binary: `clang-format -i --style=file "$f"`.

## Building

Never build on the host. `<leader>cb` in nvim-ide runs `:Build`, which is ninja in the dev
container; from a shell the equivalent is:

```bash
~/.config/nvim-ide/bin/build-devc /workspace/<project>/build/<config> ninja [target]
```

- Host `~/projects` is mounted at `/workspace` in container `alma`.
- The build directory is the newest `<project>/build/*/` holding a `compile_commands.json`.
- After changing dependencies, or when the build directory is not configured yet, run the
  configure-and-build target from the project root instead:
  `~/.config/nvim-ide/bin/build-devc /workspace/<project> make all`
