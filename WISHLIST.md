# Plugin Testing Use Cases

## What main use cases should a testing tool for Cordova satisfy?

 - enable an individual plugin developer to:
   - run javascript unit tests
   - run appium end-to-end tests
 - provide a complimentary API for instrumentation into CI

## What testing environments should such a tool target?

As many platforms that Cordova supports as possible, of course! To be more
precise:

 - running locally on:
   - a simulator or emulator
   - a real device
 - running remotely on services such as Sauce Labs

# Architecture

@purplecabbage has suggested a middleware architecture. This architecture is
usually employed around a data pipeline problem: some data passes through
several processes of transformation before being considered "complete". An HTTP
request/response pipeline is a good example: a request comes in, and a
response is returned. Several different processes, or components, may affect
the final response returned. Example components are authentication, logging, or
request validation.

In paramedic's case, what would be the data being passed through the different
layers?

 - The application-under-test itself: in the unit testing vs. appium testing
   cases, a different application needs to be created. So, perhaps a component
   consuming the cordova-cli's programmatic interface could influence this data.
 - The target environment to run the tests on: local (simulator vs. real device)
   vs. remote

# Issues Identified

- CLI flags renamed to take into account the difference, or sameness, of running local vs. remote appium.
    i.e. `--saucePlatformVersion` could be changed to `--appiumPlatformVersion` or just `--platformVersion`. The
    platformVersion capability applies to _appium_, and not _just_ to Sauce Labs.
- the local-on-appium simulator vs. real device scenario is messed up on iOS. appium needs to be set up in a special way (need a release build, ios proxy needs to run, udid of device needs to be provided).
    additionally, ios 10 requires to have signing by default. so need to pass in buildConfig flag (to specify cert / provisioning profile / dev team id details).
    paramedic should not uninstall app via xcrun simctl in local ios real device situation.
- for remote sauce labs runs, providing a `build` appium capability that aggregates all test runs from a single 'build'. Naming the build according to the pull request / periodic build names would be helpful for sorting through remote sauce runs.
- for remote sauce labs runs, providing a more descriptive `name` capability better reflecting the test behaviour.
- for CI + remote sauce labs runs, better logging out of links to sauce labs test runs
- do we run all appium tests in a single appium session? should we? it looks like errors destroy a session, which could lead to different sessions / different behaviours depending on failure modes / frequencies.
- a nicer usage help text :D
- clean up the logging
- take-screenshot-on-failure: these would have different behaviours in the remote-on-sauce vs. local-on-appium cases.
    remotely on sauce, issuing a WebDriver /screenshot command is all you need. perhaps linking to the screenshot in the job logs on failure?
    locally on appium, you'd need to issue the screenshot command and also write it out to a local temporary location. again, log out the location of the screenshot.
    is this a good use case for middleware approach?
- ability to specify a pre-built, paramedic-friendly app to use with either autotests or appium tests (to avoid having to recompile on every run, if, for example, one is just tweaking tests).
