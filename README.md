# Reproduce a genebank's lot-release practice from its day book

A harbor-format benchmark task, authored for the AfterQuery Experts programme as data
annotation for AI evaluation. Task slug `afterquery/name-released-lots` — **Science / Biology**.

An AI agent gets a container, this brief and nothing else. It works alone for
4 hours. Then a sealed verifier it never sees scores the result `0` or `1`.

## What it asks for

A seed genebank keeps its movements in a day book and answers requests for material out of it. A campaign is one such book: seed deposited, germination tests recorded, lots split, lots topped up out of one another, lots grown out again, seed withdrawn and lots discarded, all set down in the order they happened. Among them are requests, each naming an accession and a day, and for every request the store releases one of its lots. Which lot it releases is the question here, and the store's practice has to be reproduced as a program rather than worked out by hand.

The full brief is in [`instruction.md`](instruction.md).

| | |
| --- | --- |
| **Deliverable** | `/app/releases.py` |
| **Agent budget** | 4 hours (14400s) |
| **Expert estimate** | 6 hours |
| **Machine** | 1 CPU, 2048 MB RAM, 0 GPU |
| **Verifier budget** | 900s |
| **Tags** | `genebank`, `seed-store`, `state-machine`, `rulebook`, `python` |

## Why it is hard

Who does this: a genebank curator, or whoever in a seed store works the day book and decides what goes out against a request. The job is to reproduce the store's own practice as a program, not to read a figure out of a file. What is asked for: a program, `/app/releases.py`, given one campaign as JSON on standard input and printing, for each request in it, the name of the lot the store releases.

<details>
<summary>Full note</summary>

It is marked by being run on 30 campaigns rebuilt in the verifier from a seed that never ships, 289 requests in all. The answer is a NAME, so there is no quantity to scale and nothing a wrong reading can be fitted back into. What ships: `/app/GENEBANK.md`, which sets out the whole of the store's practice, and `/app/worked.json` - 14 worked campaigns, 228 requests, each carrying the name the store gave. Nothing is concealed; everything needed is derivable from what ships. Where the difficulty is. The note states the practice in full except for 5 matters it openly admits the store has never written down - whether a lot holding exactly the hundred may go, whether a lot split off carries the date of the test it inherits, whether a test dated the day of a request is in effect for it, and so on. Those make a grid of readings. Set against them is one sentence the note does not flag at all: of the lots it can release, the store releases the oldest seed. Read naturally, that is the oldest lot - the day the lot itself came into being, which every engine has to hand. Read as the store works, it is the age of the SEED: seed keeps the day it was first deposited wherever it goes afterwards, so a lot made up by topping one lot from another is as old as the older part of it, and a lot split off is as old as its parent's seed however new the lot is. Nothing in a correct engine keeps that date unless the rule has been understood, and because it orders the candidates it decides every request rather than a handful near a boundary. The whole grid is 96 readings. **Exactly one of them reproduces all 228 worked answers.** The measured ladder, at 10 points a request, top score 2890: the reference 2890 (100%) every reading going by the age of the seed 2870..2890 the bar 2790 every reading going by the age of the lot 2680..2700 naming the commonest lot every time 610 Nothing lands between 2700 and 2870. So all 5 signposted matters can be read wrongly and the task is still passed, and the one unsignposted reading decides it on its own. tests/rival_readings.py walks the grid and asserts every line of that table.

</details>

## How it is verified

tests/grader.py rebuilds the 30 graded campaigns from a seed held only in the verifier image and runs the agent's `/app/releases.py` as a subprocess, one campaign at a time, with the campaign on standard input. Nothing of the agent's is imported into the marking process, and the graded campaigns never reach the agent's container. Each request is worth 10 when the lot named is the one the store released and nothing otherwise, giving a top score of 2890; the reward is 1 at 2790 and 0 below it.

<details>
<summary>Full note</summary>

Partial credit is deliberate, so that a slip on one campaign cannot decide the reward, and the bar sits in an empty band 2700 to 2870 which the measured ladder shows nothing occupies - it separates the one reading that matters from every other, and from no other. A program that crashes, hangs beyond 60 seconds on a campaign, or prints nothing scores nothing for that campaign. An empty container scores 0 and the reference scores 2890. The generator is seeded and ships in /tests, so every figure above can be reproduced there.

</details>

<details>
<summary>Reference approach (spoiler)</summary>

solution/releases.py is standard-library Python, about a hundred lines. It plays the campaign forward, keeping for each lot its accession, its seeds, its last germination figure with the day it was made, whether it is still in the store, whether it is under hold, and the day the oldest seed now in it was first deposited. A split hands the parent's figure, test date and seed date to the child; a top-up weights the two figures by the seeds each part brings and takes the older of both the test dates and the seed dates; a regeneration starts a new lot on its own figure. At a request it takes the lots of that accession still in the store, not under hold, holding at least a hundred seeds and tested within 365 days, and names the one whose seed is oldest, breaking a tie by name. It scores 2890 of 2890.

</details>

## Layout

```
instruction.md          the brief - the only thing the agent is told
task.toml               artifacts, timeouts, resources, metadata
environment/            Dockerfile and data for the agent's container
solution/solve.sh       reference solution; scores 1 by construction
tests/                  verifier image, grading logic and ground truth
```

## Running it

With [harbor](https://github.com/laude-institute/harbor) installed and Docker running:

```bash
uv tool install harbor

harbor run -p . -a oracle -e docker   # reference solution - must score 1
harbor run -p . -a nop    -e docker   # nothing happens    - must score 0
```

Those two runs are the calibration the bundle has to satisfy: the oracle run substitutes
`solution/solve.sh` for the agent and must score `1`, the nop run does nothing and must
score `0`. Anything in between means the verifier is measuring the wrong thing.

## Reading this

`tests/` and `solution/` hold the answers. To try the task honestly, read only
`instruction.md` and `environment/`.

---

Part of a set of 38 tasks indexed at
[afterquery-data-annotation](https://github.com/helexdev1004/afterquery-data-annotation).
