# How to publish a Blazor app to github pages.
I started from these 2 tutorials:
- [https://www.davideguida.com/how-to-deploy-blazor-webassembly-on-github-pages-using-github-actions/]
- [https://www.meziantou.net/publishing-a-blazor-webassembly-application-to-github-pages.htm]

But I could not run it with both, but a mix of both.

What you need to do:
## Github:
1. Create a github repository. It can have any name: such as `MyBlazorOnGitHubPages`. We will use this name later on.
2. Go to [https://github.com/settings/tokens] and create a token. Name it whatever you want. Add `repo` to the scopes. Copy it to your clipboard.
3. Go to your repository: `Settings` > `Secrets` and create a secret called `PUBLISH_TOKEN` and paste your token.
4. There is an additional step in the end (Set github pages to use the `gh-pages` branch, on the root.) 

## Local:
1. Create a `Blazor App` Make it a `Blazor Webassembly App`. Do NOT check `ASP.NET Core hosted`. (Not sure about `PWA`, I did not try that). You can add this project into an existing solution, that makes no difference.
2. Add the following javascript code in the `body` of your `wwwroot/index.html`

``` html
<!-- Inside your body tag, after the <app> -->
<!-- Start Single Page Apps for GitHub Pages -->
<script type="text/javascript">
    // Single Page Apps for GitHub Pages
    // https://github.com/rafrex/spa-github-pages
    // Copyright (c) 2016 Rafael Pedicini, licensed under the MIT License
    // ----------------------------------------------------------------------
    // This script checks to see if a redirect is present in the query string
    // and converts it back into the correct url and adds it to the
    // browser's history using window.history.replaceState(...),
    // which won't cause the browser to attempt to load the new url.
    // When the single page app is loaded further down in this file,
    // the correct url will be waiting in the browser's history for
    // the single page app to route accordingly.
    (function (l) {
        if (l.search) {
            var q = {};
            l.search.slice(1).split('&').forEach(function (v) {
                var a = v.split('=');
                q[a[0]] = a.slice(1).join('=').replace(/~and~/g, '&');
            });
            if (q.p !== undefined) {
                window.history.replaceState(null, null,
                    l.pathname.slice(0, -1) + (q.p || '') +
                    (q.q ? ('?' + q.q) : '') +
                    l.hash
                );
            }
        }
    }(window.location))
</script>
<!-- End Single Page Apps for GitHub Pages -->
```

3. Now in your project folder: in your `wwwroot/index.html` change the base `<base href = "/" />` to `<base href="/MyBlazorOnGitHubPages/" />` where `MyBlazorOnGitHubPages` is your repository name that we used before.
4. Add the following `wwwroot/404.html` file:

``` html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Single Page Apps for GitHub Pages</title>
    <script type="text/javascript">
        // Single Page Apps for GitHub Pages
        // https://github.com/rafrex/spa-github-pages
        // Copyright (c) 2016 Rafael Pedicini, licensed under the MIT License
        // ----------------------------------------------------------------------
        // This script takes the current url and converts the path and query
        // string into just a query string, and then redirects the browser
        // to the new url with only a query string and hash fragment,
        // e.g. http://www.foo.tld/one/two?a=b&c=d#qwe, becomes
        // http://www.foo.tld/?p=/one/two&q=a=b~and~c=d#qwe
        // Note: this 404.html file must be at least 512 bytes for it to work
        // with Internet Explorer (it is currently > 512 bytes)
        // If you're creating a Project Pages site and NOT using a custom domain,
        // then set segmentCount to 1 (enterprise users may need to set it to > 1).
        // This way the code will only replace the route part of the path, and not
        // the real directory in which the app resides, for example:
        // https://username.github.io/repo-name/one/two?a=b&c=d#qwe becomes
        // https://username.github.io/repo-name/?p=/one/two&q=a=b~and~c=d#qwe
        // Otherwise, leave segmentCount as 0.
        var segmentCount = 0;
        var l = window.location;
        l.replace(
            l.protocol + '//' + l.hostname + (l.port ? ':' + l.port : '') +
            l.pathname.split('/').slice(0, 1 + segmentCount).join('/') + '/?p=/' +
            l.pathname.slice(1).split('/').slice(segmentCount).join('/').replace(/&/g, '~and~') +
            (l.search ? '&q=' + l.search.slice(1).replace(/&/g, '~and~') : '') +
            l.hash
        );
    </script>
</head>
<body>
</body>
</html>
```

5. Add a blank `wwwroot/.nojekyll` file
6.  Create a `.github/workflows/gh-pages.yml` file with the following content:

``` yml
name: 'Publish application'
on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout 🛎️
        uses: actions/checkout@v2 # If you're using actions/checkout@v2 you must set persist-credentials to false in most cases for the deployment to work correctly.

      # Install .NET Core SDK
      - name: Setup .NET Core
        uses: actions/setup-dotnet@v1
        with:
          dotnet-version: 3.1.301

      # Dotnet restore
      - name: Install dependencies
        run: dotnet restore

      # Run tests
      - name: Test
        run: dotnet test --no-restore

      # Generate the website
      - name: Publish
        run: dotnet publish Path/to/your/WasmProject.csproj --no-restore --configuration Release --output build

      # Publish the website
      - name: GitHub Pages action
        # if: ${{ github.ref == 'refs/heads/master' }} # Publish only when the push is on master
        uses: JamesIves/github-pages-deploy-action@releases/v3
        with:
          ACCESS_TOKEN: ${{ secrets.PUBLISH_TOKEN }}
          BASE_BRANCH: master # The branch the action should deploy from.
          BRANCH: gh-pages # The branch the action should deploy to.
          FOLDER: build/wwwroot # The folder the action should deploy.
          SINGLE_COMMIT: true
```

8. Commit and push

## Back on Github
1. Go to `Actions` and check that the build passes.
2. Go to our repository `Settings`, `Options` and scroll down to `Github Pages`. Enable it, select the newly created `gh-pages` branch and select the `\` path.

Voilà.