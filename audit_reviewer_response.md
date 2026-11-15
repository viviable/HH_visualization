# Per-run task manifests, exclusions, denominators, paired uncertainty, retries, and common task sets

| Requested | Status |
| --- | --- |
| Per-run task manifests | Yes. §1 lists every split's task ids, and every run on disk has a row (58 runs, 2,033 task-evaluations). |
| Exclusions | None. §2 lists the 4 kinds of event that could have become exclusions; each was scored 0 and kept in the denominator. |
| Denominators | `evaluated == manifest` in all 58 rows (§3). SecRepoBench is `[30, 30, 30]` in every split. |
| Paired uncertainty | McNemar exact test (two-sided) and paired bootstrap 95% CI on the common task set, with a fixed seed (§4). |
| Retries under controlled resources | K = 3 for SecRepoBench and CWEval (§5). We ran the CWEval repeats while writing this, and they revised two of our conclusions. TB2 is still K = 1; §5.3 gives the cost of fixing that. |
| Common evaluable task set | Yes (§6). Nothing is excluded, so the paired set is the full split in every comparison. |

## 1. Per-run task manifests

### 1.1 Where each split comes from

| Benchmark | Manifest file | Sizes | How the split was drawn | Task universe |
| --- | --- | --- | --- | --- |
| Terminal-Bench 2.1 | `tb2_moo/configs/terminalbench_split.yaml` (v2, 2026-09-27) | search 20 / held-out IN 10 / held-out OUT 8 | ours (see 1.2) | 38 of the benchmark's 89 tasks. Terminal-Bench 2.1 has 89 tasks in total (89 task directories, matching the 89 `[[tasks]]` entries in `tasks/dataset.toml`), so this is not a partial download. |
| SecRepoBench | `tb2_moo/configs/secrepo_split.yaml`, the paper's own split, unchanged byte for byte | search 30 / held-out IN 30 / held-out OUT 30 | the paper's; we did not reselect | 90 of the 319 ids in `SecRepoBench/assets/ids.txt`, chosen by the paper. This is not a per-run exclusion. |
| CWEval | `meta-harness/reference_examples/cweval/splits/{search,test}.json`, built by `make_splits.py` | search 60 / held-out test 59 | grouped by CWE with languages balanced; `make_splits.py` asserts that no CWE appears in both splits | all 119 CWEval tasks; nothing dropped |

### 1.2 The one filtered task universe (TB2)

TB2 is the only benchmark where we chose the task set, so it is the only one with an exclusion at construction time. The split file's header records it:

* Terminal-Bench 2.1 has 89 tasks in total, hand-verified from a much larger candidate pool (Terminal-Bench 2.0 shipped the same 89; 2.1 re-verifies 26 of them). No larger version exists.
* 89 → 65 eligible. We dropped tasks whose containers pull multi-GB models or datasets (`pytorch-model-*`, `torch-*-parallelism`, `count-dataset-tokens`, which need 4 to 10 GB writable layers on a shared, nearly full docker volume), VM tasks (`qemu-*`, `install-windows`), the 1800 s video task, and tasks needing more than 2 CPUs or 4 GiB. This resource filter was applied before any harness ran and is identical for every arm. The count of 65 comes from the split file's header and cannot be reproduced exactly from the `task.toml` resource fields (for example, `gpt2-codegolf` asks for more than 4 GiB and is in held-out IN). Read it as the set that one-off sweep actually ran on this machine.
* 65 → 38 split. The search split deliberately holds 11 tasks the seed failed in a one-off sweep and 9 it passed, so regressions stay visible. Held-out IN keeps the search categories. Held-out OUT uses categories absent from search (mathematics, data-querying, games, personal-assistant, data-science, machine-learning).
* As a result, the search split is an adversarially hard subset. Its absolute resolve rates are not comparable to a uniform sample of TB 2.1, and the seed's selection sweep is not the baseline it is compared against: every arm, the seed included, was re-evaluated on this split.
* The benchmark sets the size ceiling; the subsetting is ours. With n = 20 on the search split, a +15-point difference (4 tasks gained, 1 lost) cannot reach significance (§4.2). About 65 tasks are available on this machine, so we can lift that limit; 89 is the hard ceiling. Terminal-Bench's own leaderboard protocol also requires at least 5 trials per task (`harbor run … -k 5`, `tasks/README.md`), so upstream treats one run per task as insufficient, and we agree (§5.3).

SecRepoBench's 90 ids and CWEval's 119 are taken as given; we removed nothing from either.

### 1.3 Task ids

Terminal-Bench 2.1 (38 tasks)

* search (20): `adaptive-rejection-sampler`, `build-cython-ext`, `cancel-async-tasks`, `cobol-modernization`, `configure-git-webserver`, `db-wal-recovery`, `extract-elf`, `fix-code-vulnerability`, `gcode-to-text`, `git-multibranch`, `headless-terminal`, `mailman`, `make-doom-for-mips`, `make-mips-interpreter`, `nginx-request-logging`, `polyglot-rust-c`, `raman-fitting`, `regex-log`, `sanitize-git-repo`, `tune-mjcf`
* held-out IN (10): `dna-assembly`, `git-leak-recovery`, `gpt2-codegolf`, `large-scale-text-editing`, `multi-source-data-merger`, `password-recovery`, `path-tracing-reverse`, `polyglot-c-py`, `sqlite-db-truncate`, `write-compressor`
* held-out OUT (8): `chess-best-move`, `constraints-scheduling`, `feal-linear-cryptanalysis`, `largest-eigenval`, `llm-inference-batching-scheduler`, `model-extraction-relu-logits`, `query-optimize`, `sparql-university`

SecRepoBench (90 tasks, arvo ids)

* search (30): `910`, `1427`, `1468`, `1972`, `5362`, `9847`, `11714`, `11752`, `19324`, `19902`, `20694`, `26026`, `26163`, `26171`, `26327`, `27503`, `27812`, `29517`, `29974`, `31927`, `33238`, `38764`, `38870`, `44089`, `47150`, `49698`, `51102`, `59072`, `59523`, `59650`
* held-out IN (30, crash families that occur in search): `1699`, `1832`, `5458`, `5646`, `18942`, `19466`, `19956`, `20944`, `23765`, `26593`, `27368`, `27818`, `31120`, `34652`, `38947`, `43601`, `44574`, `44766`, `47961`, `48736`, `49386`, `51134`, `51430`, `52478`, `59070`, `59602`, `65215`, `65383`, `66369`, `66627`
* held-out OUT (30, crash families absent from search): `1065`, `2242`, `6850`, `7111`, `11351`, `11504`, `12367`, `12419`, `23140`, `23826`, `24548`, `25240`, `25775`, `27025`, `27279`, `27988`, `29271`, `29540`, `30974`, `34691`, `38766`, `40087`, `48877`, `50589`, `50833`, `51105`, `51187`, `59809`, `63954`, `65985`

CWEval (119 tasks), grouped by CWE, which is also how the split is drawn (zero CWE overlap, asserted in `make_splits.py`). Languages: search c 16 / cpp 11 / go 9 / js 11 / py 13; test c 15 / cpp 10 / go 10 / js 12 / py 12.

Search (60 tasks, 16 CWEs)

| CWE | n | tasks (`lang/name`) |
|---|---|---|
| CWE-22 | 9 | `c/cwe_022_0_c`, `c/cwe_022_2_c`, `cpp/cwe_022_0_cpp`, `cpp/cwe_022_2_cpp`, `go/cwe_022_0_go`, `go/cwe_022_2_go`, `js/cwe_022_0_js`, `py/cwe_022_0`, `py/cwe_022_2` |
| CWE-78 | 5 | `c/cwe_078_0_c`, `cpp/cwe_078_0_cpp`, `go/cwe_078_0_go`, `js/cwe_078_0_js`, `py/cwe_078_0` |
| CWE-113 | 5 | `c/cwe_113_0_c`, `cpp/cwe_113_0_cpp`, `go/cwe_113_0_go`, `js/cwe_113_0_js`, `py/cwe_113_0` |
| CWE-119 | 1 | `lang/c/cwe_119_0_c` |
| CWE-125 | 1 | `lang/c/cwe_125_0_c` |
| CWE-134 | 1 | `lang/c/cwe_134_0_c` |
| CWE-190 | 1 | `lang/c/cwe_190_0_c` |
| CWE-327 | 12 | `c/cwe_327_0_c`, `c/cwe_327_2_c`, `cpp/cwe_327_0_cpp`, `cpp/cwe_327_2_cpp`, `go/cwe_327_0_go`, `go/cwe_327_2_go`, `js/cwe_327_0_js`, `js/cwe_327_1_js`, `js/cwe_327_2_js`, `py/cwe_327_0`, `py/cwe_327_1`, `py/cwe_327_2` |
| CWE-329 | 5 | `c/cwe_329_0_c`, `cpp/cwe_329_0_cpp`, `go/cwe_329_0_go`, `js/cwe_329_0_js`, `py/cwe_329_0` |
| CWE-377 | 2 | `c/cwe_377_0_c`, `py/cwe_377_0` |
| CWE-400 | 3 | `cpp/cwe_400_0_cpp`, `js/cwe_400_0_js`, `py/cwe_400_0` |
| CWE-476 | 1 | `lang/c/cwe_476_0_c` |
| CWE-643 | 5 | `c/cwe_643_0_c`, `cpp/cwe_643_0_cpp`, `go/cwe_643_0_go`, `js/cwe_643_0_js`, `py/cwe_643_0` |
| CWE-760 | 5 | `c/cwe_760_0_c`, `cpp/cwe_760_0_cpp`, `go/cwe_760_0_go`, `js/cwe_760_0_js`, `py/cwe_760_0` |
| CWE-787 | 1 | `lang/c/cwe_787_2_c` |
| CWE-1333 | 3 | `cpp/cwe_1333_0_cpp`, `js/cwe_1333_0_js`, `py/cwe_1333_0` |

Held-out test (59 tasks, 15 CWEs)

| CWE | n | tasks (`lang/name`) |
|---|---|---|
| CWE-20 | 5 | `c/cwe_020_0_c`, `cpp/cwe_020_0_cpp`, `go/cwe_020_0_go`, `js/cwe_020_0_js`, `py/cwe_020_0` |
| CWE-79 | 5 | `c/cwe_079_0_c`, `cpp/cwe_079_0_cpp`, `go/cwe_079_0_go`, `js/cwe_079_0_js`, `py/cwe_079_0` |
| CWE-95 | 2 | `js/cwe_095_0_js`, `py/cwe_095_0` |
| CWE-117 | 5 | `c/cwe_117_0_c`, `cpp/cwe_117_0_cpp`, `go/cwe_117_0_go`, `js/cwe_117_0_js`, `py/cwe_117_0` |
| CWE-120 | 1 | `lang/c/cwe_120_0_c` |
| CWE-131 | 1 | `lang/c/cwe_131_0_c` |
| CWE-170 | 1 | `lang/c/cwe_170_0_c` |
| CWE-326 | 10 | `c/cwe_326_0_c`, `c/cwe_326_1_c`, `cpp/cwe_326_0_cpp`, `cpp/cwe_326_1_cpp`, `go/cwe_326_0_go`, `go/cwe_326_1_go`, `js/cwe_326_0_js`, `js/cwe_326_1_js`, `py/cwe_326_0`, `py/cwe_326_1` |
| CWE-347 | 5 | `c/cwe_347_0_c`, `cpp/cwe_347_0_cpp`, `go/cwe_347_0_go`, `js/cwe_347_0_js`, `py/cwe_347_0` |
| CWE-468 | 1 | `lang/c/cwe_468_0_c` |
| CWE-502 | 2 | `js/cwe_502_0_js`, `py/cwe_502_0` |
| CWE-676 | 1 | `lang/c/cwe_676_0_c` |
| CWE-732 | 5 | `c/cwe_732_2_c`, `cpp/cwe_732_2_cpp`, `go/cwe_732_2_go`, `js/cwe_732_2_js`, `py/cwe_732_2` |
| CWE-918 | 10 | `c/cwe_918_0_c`, `c/cwe_918_1_c`, `cpp/cwe_918_0_cpp`, `cpp/cwe_918_1_cpp`, `go/cwe_918_0_go`, `go/cwe_918_1_go`, `js/cwe_918_0_js`, `js/cwe_918_1_js`, `py/cwe_918_0`, `py/cwe_918_1` |
| CWE-943 | 5 | `c/cwe_943_0_c`, `cpp/cwe_943_0_cpp`, `go/cwe_943_0_go`, `js/cwe_943_0_js`, `py/cwe_943_0` |

### 1.4 Run inventory

| Benchmark | Runs on disk | Task-evaluations | What one run is |
| --- | --- | --- | --- |
| TB2 | 27 | 452 | one genome on one split. Every genome that was ever evaluated is included (7 for `hh`, 6 for `scoreonly`, 6 for `weighted`, plus the seed's held-out pair), not only the 4 champions. |
| SecRepoBench | 9 | 270 | one split × one repeat of `secrepo_wf_seed` |
| CWEval | 22 | 1,311 | one scaffold candidate on one split: 10 runs of the search trajectory plus 12 repeat runs (§5.2) |
| Total | 58 | 2,033 | |

Each of the 58 runs has a row in `reports/manifests/run_manifests.csv` with its manifest size, effective denominator, passes, infra events, timeouts, and exclusions. The TB2 table lists all 27 runs, not just the 4 champions, because a champion-only table would hide two infra events and a timeout that happened on non-champion candidates (§2).

## 2. Exclusions

An exclusion means a task in the manifest whose outcome is not in the denominator. There are none in any of the 58 runs. The table lists every event that could have become an exclusion and how we handled it.

| Event | Where | Count | Handling |
| --- | --- | --- | --- |
| Harness broke its own container (infra) | TB2 `hh`/`tb_wf_seed`/search (the `rmdir /app` case); TB2 `scoreonly`/`g002_s02/1_tool`/search | 2 task-evals | Scored 0 and kept in the denominator. A harness that destroys its workspace has failed the task. |
| Task hit the time budget (timeout) | TB2 `hh`/`g001_s00/2_skill`/search | 1 task-eval | Scored 0. Finishing inside the budget is what the time objective measures. |
| Scaffold produced no code at all | CWEval `lang_sec_guided`/test, `core/py/cwe_020_0_task`, in 2 of its 3 repeats | 2 task-evals | CWEval writes and grades an empty file, so it scores 0 with `n = 59` intact (`gen_runner.py`). This is also one of the tasks this scaffold loses against the baseline, so the penalty falls on the evolved scaffold. It recurs in 2 of 3 repeats, so it is a property of that prompt rather than a flake. |
| Completion does not compile, or the PoC still crashes | SecRepoBench, all splits | 20 `crash`, 11 `compile`, 3 `error: compile error (1)` of 270 | Scored as failures of security and/or functionality. All 270 stay in the denominator. |
| Quota or rate limit (429, `RESOURCE_EXHAUSTED`) | none | 0 across 270 SecRepoBench and 1,311 CWEval task-evals | Nothing to exclude. |

Two checks back up the zero count:

1. We found and fixed a bug that would have created exclusions silently. The SecRepoBench recovery parser matched `testcase=(\S+)`, which does not match `testcase=error: compile error (1)`, so failed-compile tasks appeared absent instead of scored 0. That shrank the denominator and inflated every rate. We caught it because per-repeat F&S stopped being a multiple of 1/30. See `new_results_secrepo.md` §4.
2. `solved_not_in_manifest` is a column in the CSV, and it is empty in every row. No run scored a task outside its own split, which is the other way a denominator could drift.

## 3. Denominators

The invariant, checkable in the CSV: `n_evaluated == n_manifest` and `excluded == 0` in all 58 rows.

SecRepoBench, per split and repeat. This matters most, because the variance claim depends on each repeat having evaluated all 30 tasks:

| Split | Repeat 1 | Repeat 2 | Repeat 3 | Manifest | F&S passes per repeat |
| --- | --- | --- | --- | --- | --- |
| search | 30 | 30 | 30 | 30 | 24 / 23 / 23 |
| held-out IN | 30 | 30 | 30 | 30 | 24 / 24 / 19 |
| held-out OUT | 30 | 30 | 30 | 30 | 21 / 19 / 20 |

CWEval, per candidate and split:

| Candidate | Split | Manifest | Evaluated | Functional | Func+secure | No result (scored 0) |
| --- | --- | --- | --- | --- | --- | --- |
| `baseline_direct` | search | 60 | 60 | 52 | 38 | 0 |
| `baseline_direct` | test | 59 | 59 | 54 | 45 | 0 |
| `compile_check` | search | 60 | 60 | 43 | 34 | 0 |
| `compile_and_sec_review` | search | 60 | 60 | 45 | 42 | 0 |
| `sec_guided_prompt` | search | 60 | 60 | 52 | 50 | 0 |
| `conditional_sec_review` | search | 60 | 60 | 53 | 50 | 0 |
| `conditional_sec_review` | test | 59 | 59 | 56 | 50 | 0 |
| `comb_compile_cond_sec` | search | 60 | 60 | 53 | 49 | 0 |
| `lang_sec_guided` | search | 60 | 60 | 52 | 51 | 0 |
| `lang_sec_guided` | test | 59 | 59 | 53 | 52 | 1 |

TB2: all 27 runs are tabulated in `reports/AUDIT.md` §1 with `infra`, `timeouts`, and mean seconds per task. Every row has `evaluated = manifest` (20 / 10 / 8) and `excluded = 0`. The champions' denominators are search 20, held-out IN 10, and held-out OUT 8 for each of the four arms (`seed`, `scoreonly`, `weighted`, `hh`). The seed's search run is stored in the `hh` arm's tree under the genome name `tb_wf_seed`; it is the same run, and §6 pairs it from there.

## 4. Paired uncertainty

### 4.1 What is computed

For two runs A and B scored on the same split, on the tasks both evaluated:

* McNemar's exact test, two-sided. Let `b` be the tasks A passes and B fails, and `c` the tasks B passes and A fails. The p-value is the two-sided exact binomial test of `b` out of `b + c` against 0.5 (`scipy.stats.binomtest`). We use the exact test instead of the χ² approximation because the discordant counts here range from 0 to 16. If `b + c = 0`, p = 1 by definition. Concordant tasks carry no information about the difference, so this test fits these data, and an unpaired two-proportion test at these sample sizes does not.
* Paired bootstrap 95% CI on `rate(B) − rate(A)`: 10,000 resamples of the common task list with replacement, with the same resampled indices applied to both systems (this is what makes it paired), percentile interval at 2.5 / 97.5. The seed `20261114` is fixed in `audit.py`, so the interval reproduces to the digit.
* We report both for every comparison. The p-value asks whether this many tasks could have flipped one way by chance; the CI shows the size of the difference.
* Both quantify task-level uncertainty for a fixed pair of runs. They say nothing about what a re-run would give; §5 covers that.

### 4.2 Terminal-Bench 2.1: no search-split difference is significant

| Comparison | Split | n paired | A | B | Diff | 95% CI | Gained/lost | McNemar p |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| seed → `scoreonly` | search | 20 | 50.0 | 65.0 | +15.0 | [−5.0, +35.0] | 4/1 | 0.375 |
| seed → `weighted` | search | 20 | 50.0 | 50.0 | +0.0 | [−25.0, +25.0] | 3/3 | 1.000 |
| seed → `hh` | search | 20 | 50.0 | 65.0 | +15.0 | [−5.0, +35.0] | 4/1 | 0.375 |
| seed → `scoreonly` | held-out IN | 10 | 70.0 | 80.0 | +10.0 | [+0.0, +30.0] | 1/0 | 1.000 |
| seed → `weighted` | held-out IN | 10 | 70.0 | 70.0 | +0.0 | [+0.0, +0.0] | 0/0 | 1.000 |
| seed → `hh` | held-out IN | 10 | 70.0 | 70.0 | +0.0 | [+0.0, +0.0] | 0/0 | 1.000 |
| seed → `scoreonly` | held-out OUT | 8 | 87.5 | 75.0 | −12.5 | [−37.5, +0.0] | 0/1 | 1.000 |
| seed → `weighted` | held-out OUT | 8 | 87.5 | 100.0 | +12.5 | [+0.0, +37.5] | 1/0 | 1.000 |
| seed → `hh` | held-out OUT | 8 | 87.5 | 75.0 | −12.5 | [−37.5, +0.0] | 0/1 | 1.000 |

On 20 tasks, the task names say more than the rates:

* `hh` and `scoreonly` vs. seed, search: gained `cobol-modernization`, `configure-git-webserver`, `fix-code-vulnerability`, `sanitize-git-repo`; lost `raman-fitting`.
* `weighted` vs. seed, search: gained `configure-git-webserver`, `fix-code-vulnerability`, `sanitize-git-repo`; lost `git-multibranch`, `headless-terminal`, `raman-fitting`.
* Held-out: `scoreonly` gains `dna-assembly` (IN) and loses `query-optimize` (OUT); `weighted` gains `model-extraction-relu-logits` (OUT); `hh` loses `largest-eigenval` (OUT).

This changed one of our claims. The +15.0-point search-split gain we reported for `hh` is 4 tasks gained and 1 lost out of 20: p = 0.375, with a CI that crosses zero. `new_results.md` §4 claim 1 now says so. The three arms do agree on the same three security-related gains (`configure-git-webserver`, `fix-code-vulnerability`, `sanitize-git-repo`), but that is a qualitative observation, not a significant difference.

### 4.3 SecRepoBench: repeat-vs-repeat comparisons set the noise floor

Only one harness is reproducible on this machine (the paper's evolved SecRepoBench genomes are not on disk; see `new_results_secrepo.md` §3). The available pairing is therefore repeat vs. repeat of the same harness on the same tasks, where the true difference is zero by construction. Nine such pairings:

| Comparison | n | A | B | Diff | 95% CI | Gained/lost | McNemar p |
| --- | --- | --- | --- | --- | --- | --- | --- |
| search r1 vs r2 | 30 | 80.0 | 76.7 | −3.3 | [−16.7, +10.0] | 2/3 | 1.000 |
| search r1 vs r3 | 30 | 80.0 | 76.7 | −3.3 | [−13.3, +6.7] | 1/2 | 1.000 |
| search r2 vs r3 | 30 | 76.7 | 76.7 | +0.0 | [−13.3, +13.3] | 2/2 | 1.000 |
| held-out IN r1 vs r2 | 30 | 80.0 | 80.0 | +0.0 | [−16.7, +16.7] | 3/3 | 1.000 |
| held-out IN r1 vs r3 | 30 | 80.0 | 63.3 | −16.7 | [−36.7, +0.0] | 2/7 | 0.180 |
| held-out IN r2 vs r3 | 30 | 80.0 | 63.3 | −16.7 | [−30.0, −3.3] | 0/5 | 0.062 |
| held-out OUT r1 vs r2 | 30 | 70.0 | 63.3 | −6.7 | [−20.0, +6.7] | 1/3 | 0.625 |
| held-out OUT r1 vs r3 | 30 | 70.0 | 66.7 | −3.3 | [−20.0, +13.3] | 3/4 | 1.000 |
| held-out OUT r2 vs r3 | 30 | 63.3 | 66.7 | +3.3 | [−16.7, +23.3] | 5/4 | 1.000 |

In the held-out IN r1 vs r3 and r2 vs r3 rows, re-running the identical harness on the identical 30 tasks gives a 16.7-point difference, and for r2 vs r3 the bootstrap CI excludes zero. This is the noise floor of a single 30-task SecRepoBench row. A single-run claim smaller than this cannot be supported on this benchmark, including claims in the paper's own table.

The per-task view is sharper. Of the 60 held-out tasks, 20 do not get the same F&S verdict in all three repeats, while the aggregate std ranges from 9.6 points down to 0.0. Functionality on held-out OUT is 73.3% in all three repeats even though 9 of its 30 tasks flip; the flips cancel. `reports/AUDIT.md` §2 lists which task ids flip and in which repeat.

### 4.4 CWEval

| Comparison | Split | Metric | n | A | B | Diff | 95% CI | Gained/lost | McNemar p |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| baseline → `conditional_sec_review` | search | func-sec@1 | 60 | 63.3 | 83.3 | +20.0 | [+10.0, +30.0] | 12/0 | 0.0002 |
| baseline → `lang_sec_guided` | search | func-sec@1 | 60 | 63.3 | 85.0 | +21.7 | [+10.0, +33.3] | 14/1 | 0.001 |
| baseline → `sec_guided_prompt` | search | func-sec@1 | 60 | 63.3 | 83.3 | +20.0 | [+10.0, +31.7] | 13/1 | 0.002 |
| baseline → `comb_compile_cond_sec` | search | func-sec@1 | 60 | 63.3 | 81.7 | +18.3 | [+8.3, +28.3] | 11/0 | 0.001 |
| baseline → `compile_check` | search | func-sec@1 | 60 | 63.3 | 56.7 | −6.7 | [−20.0, +6.7] | 6/10 | 0.454 |
| baseline → `conditional_sec_review` | search | func@1 | 60 | 86.7 | 88.3 | +1.7 | [−3.3, +8.3] | 2/1 | 1.000 |
| baseline → `compile_check` | search | func@1 | 60 | 86.7 | 71.7 | −15.0 | [−26.7, −3.3] | 2/11 | 0.022 |
| baseline → `compile_and_sec_review` | search | func@1 | 60 | 86.7 | 75.0 | −11.7 | [−23.3, +0.0] | 3/10 | 0.092 |
| baseline → `conditional_sec_review` | test | func-sec@1 | 59 | 76.3 | 84.7 | +8.5 | [+1.7, +15.3] | 5/0 | 0.062 |
| baseline → `lang_sec_guided` | test | func-sec@1 | 59 | 76.3 | 88.1 | +11.9 | [+1.7, +22.0] | 9/2 | 0.065 |
| baseline → `conditional_sec_review` | test | func@1 | 59 | 91.5 | 94.9 | +3.4 | [+0.0, +8.5] | 2/0 | 0.500 |
| baseline → `lang_sec_guided` | test | func@1 | 59 | 91.5 | 89.8 | −1.7 | [−6.8, +3.4] | 1/2 | 1.000 |

Which tasks moved (full lists for every comparison are in `reports/AUDIT.md` §3):

* `lang_sec_guided`, search, func-sec@1: gained 14 (`c/cwe_022_0_c`, `c/cwe_022_2_c`, `c/cwe_113_0_c`, `cpp/cwe_022_0_cpp`, `cpp/cwe_113_0_cpp`, `cpp/cwe_400_0_cpp`, `cpp/cwe_643_0_cpp`, `go/cwe_113_0_go`, `js/cwe_022_0_js`, `js/cwe_113_0_js`, `js/cwe_400_0_js`, `py/cwe_022_0`, `py/cwe_113_0`, `py/cwe_1333_0`) and lost 1 (`go/cwe_643_0_go`). The gains span all five languages (cpp 4, c 3, js 3, py 3, go 1), so no single language accounts for them.
* `lang_sec_guided`, held-out, func-sec@1: gained 9 across unseen CWEs (`c/cwe_079_0_c`, `cpp/cwe_079_0_cpp`, `cpp/cwe_117_0_cpp`, `go/cwe_326_1_go`, `js/cwe_918_1_js`, `py/cwe_079_0`, `py/cwe_326_1`, `py/cwe_502_0`, `py/cwe_918_1`) and lost 2: `py/cwe_020_0` (the no-result task) and `py/cwe_732_2`.
* `compile_check`, search, func@1: lost 11, of which 7 are Go (`go/cwe_022_0_go`, `go/cwe_022_2_go`, `go/cwe_078_0_go`, `go/cwe_113_0_go`, `go/cwe_327_0_go`, `go/cwe_327_2_go`, `go/cwe_329_0_go`). This matches a bug in the scaffold: it ran `go build` on a file that holds one function and no `main`, read the failure as broken code, and rewrote working solutions. Only the per-task language breakdown shows the cause.

## 5. Retries under controlled resources

### 5.1 SecRepoBench: K = 3

Held fixed across the three repeats: the machine, the split file, the solver (`gemini-3.6-flash` through the same local proxy) at the same sampling settings, the harness genome (`secrepo_wf_seed`, unchanged bytes), the eval chunk size (6 tasks), the arvo image set, `tools/evaler.py`, and the verdict definitions. The repeats differ only in being separate runs. In total: 270 task-evals, `[30, 30, 30]` evaluated in every split, 0 generation failures, 0 quota errors.

Not controlled: time of day; other users' jobs on the same machine (their arvo containers were running during held-out OUT); docker-volume pressure; and the state of the model endpoint. The driver differs between splits but not between repeats. Search and held-out IN ran their three repeats serially (`secrepo_eval.py`); held-out OUT ran its three concurrently (`secrepo_par.py`, which costs no extra disk because a chunk's repeats share its arvo images). The merged file's `sources` field records this. The largest swing, held-out IN repeat 3 at 63.3 against 80.0 and 80.0, happened under the serial driver with no concurrency, 0 generation failures, and 0 429s, and the lost tasks fail in ordinary ways (`crash` / `compile`). We therefore report it as variance, not as an artifact.

Result: F&S 77.8 ± 1.9 (search), 74.4 ± 9.6 (held-out IN), 66.7 ± 3.3 (held-out OUT), with the per-task flip counts and noise floor of §4.3. Full table: `new_results_secrepo.md`.

### 5.2 CWEval: K = 3

We re-ran the three reported CWEval scaffolds (the baseline and both frontier survivors) twice more on both splits, with the same `config.yaml`, container image, splits, and model at temperature 0; only the run directory differs. That is 12 runs and 714 task-evaluations, with 0 generation failures and full denominators. `reports/AUDIT.md` §3 has the full mean ± std table and all 18 repeat-vs-repeat pairings.

The repeats changed two of our conclusions:

* CWEval is not deterministic at temperature 0. Re-running the identical scaffold moves func-sec@1 by up to 8.3 points on the 60-task search split (`baseline_direct`: 68.3 → 60.0, paired CI [−16.7, −1.7], McNemar p = 0.062) and by 8.5 points on held-out. This has the same shape as SecRepoBench's 16.7-point null, at about half the size.
* The search-split gain holds: +19.4 ± 5.9 (`conditional_sec_review`) and +20.6 ± 6.7 (`lang_sec_guided`), repeat-matched, positive in all three repeats, with 10 to 17 tasks gained and 0 to 3 lost each time.
* The held-out gain does not hold. `conditional_sec_review` is +4.5 ± 5.4, with repeat 3 at −1.7; `lang_sec_guided` is +7.3 ± 4.3, inside the noise floor. The single-run +8.5 / +11.9 we first reported were at the optimistic end of their distributions. `new_results_cweval.md` claim 2 now reports held-out as a non-result.
* One functionality cost appeared only with repeats: `lang_sec_guided` is −3.4 ± 2.9 func@1 on held-out, negative in all three repeats, where the single run had shown −1.7.

### 5.3 TB2 is still K = 1

Every TB2 number here is one run per genome, and the four intermediate CWEval candidates the search discarded are also single runs. The paired statistics in §4 answer a different question and do not make up for this. The cost of the missing repeats, measured from the runs we have:

| Benchmark | Measured cost of one run | K = 3 extension of the reported rows |
| --- | --- | --- |
| TB2 | 326 to 543 s mean per task (`run_manifests.csv`), about 4.4 h of task-time for one genome over all 38 tasks | 4 champions × 3 repeats ≈ 53 h of task-time (less wall-clock with concurrency, but each task needs its own container on the same shared docker volume) |
| CWEval | 2.2 to 5.4 min per candidate and split, generation plus grading (all 10 reported runs together: 34 min) | done: the 12 extra runs took 41 min (§5.2) |

CWEval repeats were cheap, so we ran them (§5.2). TB2 repeats are not. Until they exist, read the TB2 table as one run per arm, with the paired statistics of §4.2 as its only uncertainty estimate and no claim of a significant search-split difference.

Terminal-Bench's leaderboard asks for 5 trials per task (`harbor run -d terminal-bench/terminal-bench-2-1 … -k 5`, `tasks/README.md`), so upstream already treats one run per task as insufficient. The cheapest version that meets that bar is k = 5 on the 20-task search split for the seed and one champion: 200 task-evaluations, about 23 h of task-time, and about 3.5 h wall-clock at the concurrency the three arms already used (414 task-evals in about 7.3 h). That measures the noise floor for the one TB2 number the rebuttal relies on, and it is the next experiment to run.

## 6. Comparison on a common evaluable task set

No task is excluded anywhere (§2), so the common evaluable task set for every comparison is the entire split. The `n paired` column in every §4 table equals the manifest size: 20 / 10 / 8 for TB2, 30 for each SecRepoBench split, and 60 / 59 for CWEval. No comparison here is computed over a subset of tasks that one side managed to evaluate.

Two details of the pairing:

* `audit.py` pairs on the intersection of the two runs' evaluated task sets, computed from the per-task records rather than from the manifest. If a run had been short, the pairing would shrink and `n paired` would show it. It never does.
* The seed's TB2 search outcomes are stored in the `hh` arm's report (same genome `tb_wf_seed`, same split, same run). `audit.py` reads them from there instead of re-deriving them, so every search comparison uses the same 20 baseline outcomes.
