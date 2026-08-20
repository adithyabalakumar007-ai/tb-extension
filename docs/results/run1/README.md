# Raw results behind RESULTS_RUN1.md and INTERIOR_GLARE.md

Committed because `outputs/` is gitignored and these are the only evidence for
the claims in those two documents. All four sets are CPU-only synthetic-film runs
reproducible from the commands below. Nothing here needs data or a GPU.

The Kaggle-derived numbers in `RESULTS_RUN1.md` (fiducial coverage, LOCO folds,
UQ table, conformal, certificates, clinic stats) are **not** here — those came
from a Kaggle session and live with the notebook outputs.

## `validate_physics_5seed/`

The definitive falsification run, under the derived contrasts
(`source="DERIVED-SLAB-v1"`).

    python scripts/validate_physics.py --seeds 5 --out outputs/physics_validation_derived

80 conditions = 5 seeds x 4 severities x 4 findings, size 320, 24 trials per
contrast level. This is the run behind the headline result: pooled median ratio
1.11, per-seed medians 0.45 / 1.81 / 0.82 / 1.95 / 0.96, across-seed spread
4.32x. `detectability.csv` has one row per condition with the `seed`,
`degenerate` and `unphysical` columns added by the patched gate.

`channel_recovery.csv` carries the veil and PSF errors by severity that the
interior-glare work follows up on.

## `validate_physics_seed0_nominal/` and `validate_physics_seed1_nominal/`

The two single-seed runs on the **retired NOMINAL** finding table that exposed
the censoring bug. Kept because they are the only direct evidence for it, and
because the table they used no longer exists in the code.

    python scripts/validate_physics.py --seed 0    # 16/16 conditions, median 0.45
    python scripts/validate_physics.py --seed 1    # 12/16 conditions, median 1.81

Compare `detectability.csv` between the two. Seed 1 has four rows at severity
0.75 with `dprime_slope = 0`, `empirical_threshold = inf` and a predicted floor
above 8 OD — physically impossible on film that spans roughly OD 0.2-3.0. The
pre-patch code filtered on `ratio > 0`, which dropped exactly those four rows, so
the run that failed harder reported the safer-looking median.

These were produced before the patch, so they lack the `seed`, `degenerate` and
`unphysical` columns.

## `interior_glare/`

Veil error split into the beam-stop annulus (interpolation) and the field
interior (extrapolation).

    python scripts/measure_interior_glare.py --n-images 20 --size 384 \
        --out outputs/interior_glare

20 synthetic chests x 5 severities, 80 of 100 captures usable. Columns are in
units of the median true interior signal, shared by both regions — see the metric
note in `docs/INTERIOR_GLARE.md` for why normalising by local signal is invalid
here. `err = estimated - true`, so positive means the veil is over-reported.
