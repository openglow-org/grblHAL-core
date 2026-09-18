# ForgeFIRM fork changes

This branch (`forgefirm`) is grblHAL core with the changes below. It is the
submodule `grblHAL-glowforge` pins at `src/grbl`.

Modified by 514 LLC d/b/a OpenGlow, 2026. Every upstream copyright notice is
unchanged, and no file carries an OpenGlow copyright line: these are changes to
Terje Io's work, not a new work.

Four of the five changes are general core fixes rather than Glowforge-specific
ones, and are candidates for upstream. When one is submitted it goes as the code
change alone, with authorship in git's `Author:` field and a `Signed-off-by:`
trailer -- never as an added copyright or "modified by" line in the diff.

## Changes

| File | Change |
|---|---|
| `planner.c` | `plan_reset_buffer()` linked the block ring with a byte-wide index, so a planner buffer of 255 blocks or more never finished initializing and the protocol never ran. `$398` allows 30 to 1000; the index is now `uint_fast16_t`. |
| `settings.c` | `step_us_min[]` was 4 bytes, but `hal.step_us_min` renders through `ftoa(v, 1)` and a DMA or stream step engine reports a minimum of 10 us or more ("35.5" needs 5 bytes). The overflow hit the adjacent statics: fortified builds abort in `settings_init`, plain builds silently truncate the neighbor. Sized for the format. |
| `spindle_control.h` | Adds `float rate_ratio` to `spindle_param_t`: the velocity ratio of the segment being prepared for a rate-adjusted laser block, 1.0 otherwise. |
| `stepper.c` | `st_prep_buffer()` records that ratio in the active spindle's param as it prepares each segment. A laser that shapes its output by speed needs the segment's own ratio, and nothing else within the driver's reach holds the block's programmed S -- the parser's S runs ahead of execution by the planner depth. |
| `state_machine.c` | In laser mode the segments prepared during a hold are the first ones a resume executes, and clearing the step-control flags at hold completion left them with no spindle update, so a constant-power (M3) block resumed dark for one segment buffer. Hold completion now resets the rpm cache and sets the update flag. |

`AGENTS.md` is OpenGlow's own file and is not a modification of upstream work.

## Rebasing

The changes are small and confined to the five files above. Rebase onto upstream
`master`, then update this file if any change is reshaped, dropped because
upstream fixed it, or added.
