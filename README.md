
# test && commit || revert (TCR)

TCR scripts - the way I do it. Inspired by [Kent Beck's Limbo on the Cheap](https://medium.com/@kentbeck_7670/limbo-on-the-cheap-e4cfae840330), [Test && Commit || Revert](https://medium.com/@kentbeck_7670/test-commit-revert-870bbd756864), and [Thomas Deniffel](https://medium.com/@tdeniffel/tcr-variants-test-commit-revert-bf6bd84b17d3).

This is written for projects with a supported build wrapper, and used on MacOS.

## Workflow

`tcr` is the main command. It compiles, verifies, commits on a green run, and notifies plus reverts production-code changes on a red run.

## Scripts

- `buildIt`: Compiles production and test code through `buildTool`.
- `testIt`: Stages current changes, runs verification, and unstages changes before returning failure when verification fails.
- `buildTool`: Finds the project build wrapper and runs the matching TCR command.
- `commitIt`: Opens the commit dialog. I use [Arlo's Commit Notation](https://github.com/arlobelshee/ArlosCommitNotation/blob/master/README.md).
- `revertIt`: Preserves non-production changes, stashes reverted production changes with the `tcr-revert-backup` message, and uses `TCR_PROD_PATH` to identify production code.
- `success`: Sends a MacOS success notification.
- `failure`: Sends a MacOS failure notification and exits nonzero.
- `buddy`: Waits for a file change under `TCR_WATCH_PATH` before running `tcr`. Requires `fswatch`.
- `collab`: Repeatedly pulls with rebase/autostash, pushes only after a successful pull, then sleeps for 30 seconds.

## Build wrappers

`buildTool` auto-detects wrappers in this order:

- `./gradlew`: `testClasses` for compile, `build` for verify
- `./mvnw`: `test-compile` for compile, `verify` for verify

Set `TCR_BUILD_TOOL` to force one when a project has multiple wrappers:

```
TCR_BUILD_TOOL=maven tcr
```

## Project layout

The default layout is a JVM-style project with watched files under `src` and production code under `src/main`. Override these when a project uses another layout:

- `TCR_WATCH_PATH`: path watched by `buddy`; defaults to `src`.
- `TCR_PROD_PATH`: path treated as production code by `revertIt`; defaults to `src/main`.
- `TCR_STASH_MESSAGE`: stash message used by `revertIt`; defaults to `tcr-revert-backup`.
