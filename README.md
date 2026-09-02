# Release candidate self-repository demo

This repository proves that a workflow dispatched at a release-candidate tag
can use `$/` to run an action from that exact tagged commit.

The tagged commit contains `release-marker.txt` with `release-candidate`.
`main` advances afterward and changes the marker to `main-after-candidate`.
The validation still reads `release-candidate`, proving that `$/` follows the
workflow run's RC commit rather than the repository's current default branch.

## Run it

1. Open **Actions → Release candidate demo → Run workflow**.
2. Keep `v0.1.0-rc.1`.
3. Follow the linked validation run in the job summary.

The parent workflow cannot call a reusable workflow with a computed
`jobs.<job_id>.uses` ref because expressions are forbidden there. Instead, it
dispatches `release-test.yml` at the RC tag and waits for that exact run.
