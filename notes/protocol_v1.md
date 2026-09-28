# Protocol v1

**Date locked:** _(fill in before the main sweep)_

Settings are in `configs/protocol_v1.json` (written by the notebook). Any change after locking gets a new version number.

## Pre-set rules

- Final numbers come only from the 436-example **eval** half. The dev half is for debugging and the pilot.
- The model at the end of training is evaluated. No best-epoch selection.
- A run with eval accuracy below 60% is flagged "degenerate." It is kept and reported, never dropped.
- Failed runs are logged with the reason.

## Predictions (written before running the main grid)

- At n = 256, p = 0.3 compared with p = 0.0: _better / worse / same, by about ___ points_
- At n = 16,384, p = 0.3 compared with p = 0.0: _
- Will the pattern rise then fall, like Figure 10 in Srivastava et al. (2014)? _
- Step-matched check (n = 256 trained for 768 steps): _
