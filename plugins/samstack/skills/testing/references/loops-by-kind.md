# Loops by kind of software

Each section fills in the steps of a loop from [SKILL.md](../SKILL.md) for one kind of software. Where the repo already has a harness, use it.

## Mac app

- **Build:** the debug app, incrementally, in the worktree's own build folder (`swift build --product <App>`, or `xcodebuild build -scheme <App> -derivedDataPath <worktree build folder>`).
- **Scratch home:** set `HOME` to the scratch home. A scratch `HOME` doesn't isolate the app's defaults, so copy the built `.app` and give the copy its own `CFBundleIdentifier` with PlistBuddy (`com.example.<app>.verify`). Re-sign it ad hoc (`codesign --force --sign - --deep`) and launch that copy.
- **Seed:** write the made-up data through the app's own CLI or file format, before launch.
- **Launch:** run the binary inside the copy in the background, record its PID, and wait until its window exists and its first screen answers.
- **Drive:** by accessibility label, through a small Accessibility API (AXUIElement) helper, or through a control socket or CLI the app already exposes.
- **Evidence:** the window's accessibility outline as text, and a picture from `screencapture -l <window id>` of a titled window. A borderless window off screen captures nothing. Add the app's log, from `log show --predicate 'process == "<App>"'`.
- **Second view:** the app's CLI, or its stored file read back.
- Never drive, read or quit the copy of the app the user is running.

## iOS and iPadOS app

- **Build:** for the simulator, incrementally (`xcodebuild build -scheme <App> -destination 'platform=iOS Simulator,id=<udid>' -derivedDataPath <worktree build folder>`).
- **Scratch home:** a simulator device the loop creates for the run (`xcrun simctl create`) and deletes by UDID at cleanup (`xcrun simctl delete <udid>`). It never touches a device someone else uses, and devices don't pile up on disk.
- **Seed:** install the app, then write made-up data, produced by the app's own code, into its container (`xcrun simctl get_app_container <udid> <bundle id> data`). To prove persistence, hash the file before and after.
- **Launch:** `xcrun simctl launch <udid> <bundle id> <launch arguments>`.
- **Drive:** deep links (`xcrun simctl openurl`) and launch arguments where they reach the change. Otherwise use an XCUITest runner as the driver: it presses accessibility identifiers, saves evidence, and checks only the end state.
- **Evidence:** `xcrun simctl io <udid> screenshot`, the accessibility hierarchy from the driver, and the app's log. After any reshoot, open every picture again.
- A web page's iPhone-only behaviour is checked in the simulator's Safari; desktop WebKit is not a stand-in.
- Where a repo's CONTRIBUTING bars simulators on a machine, it names the machine its iOS loops run on.

## CLI

- **Build:** the binary, incrementally (`cargo build`, `go build -o <bin>`, `swift build --product <tool>`, `npm run build`). Run that binary, not a wrapper that rebuilds it.
- **Scratch home:** point `HOME`, `XDG_CONFIG_HOME`, `XDG_DATA_HOME` and the tool's own config variable at the scratch home.
- **Drive:** by arguments and stdin. An interactive or long-running command is driven through a PTY (`expect`, or Python's `pty`).
- **Evidence:** each command, its stdout and stderr, and its exit code.
- **Check:** the exit code, the output, and the files it wrote, read back. Include known bad input: a missing file, a malformed config, a non-number where a number goes, an unreadable path. Each must fail with a message that says what to do.

## Web app

- **Build:** the dev server's incremental build. Use the production build when the change is in bundling, env inlining or streaming, which the dev server does differently.
- **Scratch home:** a port of the run's own, and a scratch database: a fresh SQLite file, or a database or schema the loop creates and drops. Refuse to start if `DATABASE_URL` resolves to production.
- **Seed:** made-up rows in production's shape, through the app's own migrations and seed path. Copy real column types and serialised forms, never real rows.
- **Drive:** Playwright, by role, label, test id or route (a Playwright helper skill, if you have one, saves the boilerplate). Sign in as a made-up user the seed created. Check a mobile layout at a real device profile (`devices["iPhone 13"]`), not a resized desktop window.
- **Evidence:**
  - a screenshot taken after hydration settles;
  - the page's accessibility snapshot or `innerText`;
  - console errors and failed requests;
  - geometry measured with `getBoundingClientRect` through `page.evaluate`;
  - latency timed from inside the page.
- **Second view:** the API, or a database read of what the page wrote.
- A link check follows the link to its target, rather than checking that the element exists. A click goes through the browser's own hit-testing, not a synthetic DOM event.

## Service (API, worker, daemon)

- **Build:** the server binary or bundle, incrementally. When the change is in how the service starts, launch the artifact that ships, not a dev shim.
- **Scratch home:** the service's own port, database, queue and config. Third parties (payments, email, storage) point at a local sink server the loop starts, which records what it received, or at the vendor's test mode. Never a live account.
- **Drive:** real requests on the documented API, with `curl` or a short client script.
- **Evidence:** each request, its status and its body.
- **Check:** the response, and the side effect read back: the row through a second endpoint or a database read, the email in the sink's log, the job's result.
- **Workers and daemons:** enqueue made-up work and wait for the result with a deadline that fails the loop when it passes, never a fixed sleep. Where the change touches the lifecycle, stop the process mid-work, restart it, and check the work completes exactly once.
- A child process that must talk back to a server in the same process is spawned asynchronously.

## Scripts and hooks

- **Scratch home:** a temp folder, with a throwaway repo in it for anything that reads git.
- **Drive:** run the script on made-up input, with one good case and one broken case for each thing it guards.
- **Check:** the good case passes, and the broken case is refused with a message that names the fix. Read the script's own exit code, never one through a pipe.
- **Evidence:** the transcript of both runs, with their exit codes.
- Write a guard to proceed only when the good condition is provably true, so an error refuses. The shell traps are in [lessons.md](../lessons.md).

## Worked example: a Mac app's kit

A Swift Mac app keeps its loops in `scripts/verify/` and its feature map in `docs/verify/`:

- **`launch [--bin <dir>] [--run <name>]`** takes the bin folder of the debug build the pre-push hook just made, so it doesn't build again. It writes a home of made-up data (hundreds of records across several projects, no real accounts). It then starts the run's own copy of the app on that home, and waits until the window is drawn and the app's control socket answers.
- **`doctor`** prints one ok line for each precondition: the run's process, the binary launch built, the home, the socket and the window.
- **`drive`** has these steps:
  - `show <screen>` goes to a screen;
  - `press "<label>"` presses a control by its accessibility label;
  - `expect "<text>"` checks for text;
  - `shot <step>` keeps the window as it is;
  - `cli <step> -- <args>` reads the result back through the app's CLI, as the second view.
- **Evidence:** every step keeps a numbered outline, picture and log in `.scratch/verify/<run>/`. The revision and binary digest go in `run.txt`.
- **`cleanup`** stops only the run's own app and removes its home. The evidence stays.
- **Feature map:** each `docs/verify/<feature>.md` says how to reach the feature, gives the exact commands that drive it, and lists its gotchas.
