# Screenshot Testing

Unit tests are great for JavaScript. But how do you test CSS? With screenshots! We use a [headless](https://chromium.googlesource.com/chromium/src/+/lkgr/headless/README.md) instance of Chrome controlled through [Puppeteer](https://github.com/GoogleChrome/puppeteer) to generate screenshots of our test pages. Those generated screenshots are compared through [Resemble.js](https://github.com/HuddleEng/Resemble.js) for [pixel-perfect](#about-pixel-perfect-comparison) accuracy. The golden screenshots (ones deemed to be correct) are then checked in the repository, ensuring we have a reference for how a component should look.

## Testing your changes

You've made some changes to a component and want to make sure it looks the same. It's time to test your changes.

1. `$ npm start` - This starts the dev server, a necessary step to ensure our browser can open the testable webpages.
2. Open another terminal session in which we'll run our tests.
3. `$ npm run test:image-diff`

If all the tests pass, your changes are [pixel perfect](#about-pixel-perfect-comparison). Nice!

If your tests fail, then it's time to debug.

## Debugging local test failure

The golden screenshots are stored in `/test/screenshot/<component-name>` alongside the webpages they reference. The naming is as follows:

| Component name | `sample-component` |
| Test page | `http://localhost:8080/sample-component/foo.html` |
| Golden screenshot | `/test/screenshot/sample-component/foo.html.golden.png` |
| Snapshot screenshot | `/test/screenshot/sample-component/foo.html.snapshot.png` |
| Diff between golden and test | `/test/screenshot/sample-component/foo.html.diff.png` |

> **Note**: Neither the `*.snapshot.png` nor `*.diff.png` screenshots get checked in to the repo. This helps to keep the repo clean and makes it serve as the single source of truth for how components should look.

## Capturing your changes

You've made a new component or confirmed that your snapshot images are indeed how things should look. Time to capture some new goldens.

1. `$ npm start` - This starts the dev server, a necessary step to ensure our browser can open the testable webpages.
2. Open another terminal session in which we'll run our tests.
3. `$ npm run capture`

The new goldens will overwrite existing golden screenshot files and create new ones where none exist.

## Debugging CI test failure

You've made changes and opened a pull request. However, CI informs you that your changes aren't passing the screenshot test. Time to debug!

Screenshots generated through CI are uploaded to a Google Cloud Storage bucket. Each commit to a PR uploads new screenshots. They're publicly accessible through the following URL scheme:

`https://storage.googleapis.com/screenshot-image-captures/<commit-hash>/<path>/<to>/<file>.html.<type>.png`

where `<commit-hash>` is the [short eight character hash of the commit](https://stackoverflow.com/a/5694416) and `/<path>/<to>/<file>` is the path to the HTML file you wish to view, relative to the `/test/screenshot` folder.

For example, `/test/screenshot/sample-component/index.html` with commit `abcd1234` would have a diff screenshot at the following path:

`https://storage.googleapis.com/screenshot-image-captures/abcd1234/sample-component/index.html.diff.png`

>**Note**: To keep storage costs low, screenshots will need to be cleaned up periodically, perhaps through a cron job.

<span id="pixel-perfect-disclaimer"></span>

### About pixel perfect comparison

Our screenshot comparison ignores anti-aliasing differences between the golden screenshots and your snapshots. This is to ensure that OS-level differences between how subpixels are rendered don't cause test failures. It also ensures that we can have a single source of truth for the goldens instead of one golden per OS. You can read more about subpixel rendering [here](https://en.wikipedia.org/wiki/Subpixel_rendering).