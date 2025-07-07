# Update/ Possible Archival
You are likely better of using vitest if you can which I think supports what this project was doing for jest by not isolating the test suites.

# Hella fast jest runtime
This is a jest runtime that doesn't isolate test suites to increase speed dramatically in many situations (up to 8x in ours). I wrote the original version of this around 4 years ago for the startup I worked at and we saw very large performance improvements for one of our frontend apps that was quite slow especially on the old intel MacBooks. Over those 4 years there was never any weird issues that cropped up from doing this at least on our large frontend app it was used for and it seemed fairly stable. 

You could also implement this as a custom jest runner instead which would probably be more robust overall but I ran into a couple issues with that and this way just ended up being easier.

## The Problem
The core problem that this is addressing is that jest runs test suites in isolation from each other. So for every test file you have jest has to reload many modules and setup a new test env which can be very slow at least on some projects especially ones using jsdom. Running test suites in isolation is of course helpful because it helps prevent tests interfering with each other which can be a real pain to debug and track down, but I've found that this type of problem is quite rare. The trade off of having way faster tests over these occasional test bugs is way worth it to me and probably other people too.

Test frameworks I have worked with in the past such as JUnit have given you the option of choosing between running your tests in complete isolation or not (and even defaulted to this way). Jest does not provide you with such an option.

## Performance Results
Running on Neat Capital's react frontend with a M1 Max and without using jest cache by doing a `jest --clear-cache` before doing each test run:

Before:
```
Test Suites: 3 failed, 1 skipped, 1708 passed, 1711 of 1712 total
Tests:       6 failed, 8 skipped, 5699 passed, 5713 total
Snapshots:   0 total
Time:        228.658 s
```

After:
```
Test Suites: 3 failed, 1 skipped, 1708 passed, 1711 of 1712 total
Tests:       6 failed, 8 skipped, 5699 passed, 5713 total
Snapshots:   0 total
Time:        28.327 s
```

Note you won't see quite as dramatic gains all of the time because of the normal jest test cache and for running single tests you still have the initial startup time. If your project is cleaner than ours and doesn't have large imports then you might not see as much of an improvement. If you are not using the jsdom env you will probably see smaller improvements too.

## Caveats
- Files using jest.mock() to mock modules will automatically be excluded from the fast runtime and will fallback to using the slow isolated default instead. Somebody smarter than me or me in the future might be able to sort of fix this problem.
- If you are setting globals like window or document or whatever in your tests, they will break when using fast-jest-runtime since test suites are not completely isolated.
- Using timers/fakeTimers will not work (you can use `// @fast-jest-ignore` in files to work around this, and this quirk can likely be fixed in the future I forget what the problem was)
- The current version is for jest 29.7.0, slight chance it might work with older versions too though. (I've previously written a version for 24.X, if you want it DM me somewhere)
- This does not attempt to fix memory issues (it probably even makes that problem worse), if you want a slower runner but one that addresses memory issues check out this one: https://github.com/goloveychuk/fastest-jest-runner 
- If you're doing just node you might want to look at this guy: https://github.com/nicolo-ribaudo/jest-light-runner it has some unsupported jest features however.
- for multi project workspaces todo feature of making a new env/module cache per project which would probably be helpful

## Usage
`npm i @m-abboud/hella-fast-jest-runtime --save-dev`

### For frontend apps or anything using jsdom 
Add these two lines to your jest config
```
"jest": {
  testEnvironment: '@m-abboud/hella-fast-jest-runtime/lib/fastJsdomEnvironment',
  runtime: '@m-abboud/hella-fast-jest-runtime/lib/fastJestRuntime',
}
```

### For node
Add these two lines to your jest config
```
"jest": {
  testEnvironment: '@m-abboud/hella-fast-jest-runtime/lib/fastNodeEnvironment',
  runtime: '@m-abboud/hella-fast-jest-runtime/lib/fastJestRuntime',
}
```

**Note you must always use both the fast jest test env and the runtime. They will not work by themselves**

### Exclude comment
You can include a comment like this anywhere in a test file to exclude it from using the fast jest runtime and instead to fall back to the normal isolated mode. This is useful for cases where fast jest does not work or your test file is breaking other tests.
```
// @fast-jest-ignore
```
