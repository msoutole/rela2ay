# src/tui/render/

## Responsibility
View Presentation and Widget Composition Subsystem. Translates application models and domain entities into formatted Ratatui widgets, handling layout geometry calculations, theme palette application, ANSI styling, and responsive column rendering.

## Design
- **Composite Widget Pattern**: `mod.rs` orchestrates high-level layout partitioning (header, body, footer, modal layers) and delegates to specialized sub-renderers:
  - `agents.rs`: Tabular representation of agent instances (PID, state badge, tool icon, event counters).
  - `messages.rs`: Chronological message stream, threaded conversation layout, and event lines.
  - `launch.rs`: Interactive agent launch form with configurable tool and preset parameters.
  - `text.rs`: Markdown formatting, mention highlight spans (`@agent`), and ANSI color code parsing.
- **Theme Encapsulation**: Styles widgets using colors and symbols provided by `src/tui/theme.rs`.

## Flow
1. **Frame Initiation**: Terminal refresh invokes `render::draw_ui(frame, app)`.
2. **Layout Slicing**: Splits screen dimensions into constraint-based rects (vertical/horizontal splits).
3. **Widget Painting**: Draws agent lists, chat timelines, status bars, and active modal overlays into the target frame.
4. **Buffer Flushing**: Terminal driver flushes rendered frame to the physical or virtual display.

## Integration
- **Consumed by**: `src/tui/app.rs`, `src/tui/inline/eject.rs`.
- **Depends on**: `ratatui`, `src/tui/theme.rs`, `src/tui/model.rs`, `src/shared/ansi.rs`.

