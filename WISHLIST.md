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

Feedback welcome @purplecabbage / @alsorokin !

# Issues Identified / Features Wanted

- CLI flags renamed to take into account the difference, or sameness, of running local vs. remote (Sauce Labs') appium.
    i.e. `--saucePlatformVersion` could be changed to `--appiumPlatformVersion` or just `--platformVersion`. The
    platformVersion capability applies to _appium_, and not _just_ to Sauce Labs.
- the local-on-appium simulator vs. real device scenario is messed up on iOS. appium needs to be set up in a special way:
  - need a signed build, so the `cordova build` command needs to be provided with a `buildConfig` flag to specify required cert/provsioning profile, and, in the latest versions of ios, team ID details.
  - ios webkit debug proxy needs to run
  - udid of device needs to be provided
  - paramedic should not uninstall app via xcrun simctl in local ios real device situation, could use libimobiledevice for this instead.
- for remote sauce labs runs, providing a set of sauce-specific capabilities could be helpful to annotate the tests and make it easier to browse through the sauce labs test archives:
  - a `build` appium/selenium capability would aggregate multiple test runs into a single 'build'. Naming the build according to the pull request / periodic build names would be helpful and help in organization..
  - providing a more descriptive `name` appium/selenium capability better reflecting the test behaviour, would make clearer what each individual test session Sauce did.
  - using the sauce labs REST API to update, after the test run completes, whether individual tests passed or failed would further improve the sauce labs browsing experience.
- for CI + remote sauce labs runs, better logging out of links to sauce labs test runs, so we can quickly find the sauce link (currently have to infer it from webdriver session logging).
- we (try to) run all appium tests in a single appium session. should we? it looks like errors cause a 'test restart' behaviour, which destroy the current session and recreates a new one. this could lead to different sessions / different behaviours depending on failure modes / frequencies. a best practice is to run a single test case per session, ensuring full test isolation.
  - the negative in that situation is likely a higher run time on sauce labs. should measure the impact on CI run length to see if it is significant enough to deter us from this path.
  - the positive in that situation is that we can leverage parallelism to run tests in parallel, thus in theory speeding up our test runs.
    - we should see what our maximum concurrency in the sauce labs account is, and use it if it is more than one. This would significantly decrease test run times.
  - the local case should not be affected by test isolation at all.
- a nicer usage help text :D 
- clean up the logging
- take-screenshot-on-failure: these would have different behaviours in the remote-on-sauce vs. local-on-appium cases.
  - remotely on sauce, issuing a WebDriver /screenshot command is all you need. perhaps linking to the screenshot in the job logs on failure? 
  - locally on appium, you'd need to issue the screenshot command and also write it out to a local temporary location. again, log out the location of the screenshot.
- ability to specify a pre-built, paramedic-friendly app to use with either autotests or appium tests (to avoid having to recompile on every run, if, for example, one is just tweaking tests).
