<p align="center">
    <img src="https://github.com/codegouvfr/keycloakify-starter/workflows/ci/badge.svg?branch=main">
</p>

A starter/demo project for [Keycloakify](https://keycloakify.dev)

# Introduction

This repo constitutes an easily reusable setup for a standalone Keycloak theme project OR for a SPA React App that generates a
Keycloak theme that goes along with it.  
If you are only looking to create a theme (and not a theme + an App) there are a lot of things that you can remove from this starter: [Please read this section of the README](#standalone-keycloak-theme).

# Quick start

```bash
yarn
yarn build-keycloak-theme # Build the theme one time (some assets will be copied to 
              # public/keycloak_static, they are needed to dev your page outside of Keycloak)
yarn start # See the Hello World app
# Uncomment line 15 of src/keycloak-theme/kcContext, reload https://localhost:3000
# You can now develop your Login pages. (Don't forget to comment it back when you're done)

# Think your theme is ready? Run
yarn build-keycloak-theme
# Read the instruction printed on the console to see how to test
# your theme on a real Keycloak instance.
```

# The CI workflow

-   You need to manually allow GitHub Action to push on your REPO.  For this reason the initial setup will fail.  You need to enabled permission and re-run failed job: [see video](https://user-images.githubusercontent.com/6702424/213480604-0aac0ea7-487f-491d-94ae-df245b2c7ee8.mov).  
-   This CI is configured to both publish on [GitHub Pages](https://github.com/codegouvfr/keycloakify-starter/blob/3617a71deb1a6544c3584aa8d6d2241647abd48c/.github/workflows/ci.yaml#L51-L76) and on [DockerHub](https://github.com/codegouvfr/keycloakify-starter/blob/3617a71deb1a6544c3584aa8d6d2241647abd48c/.github/workflows/ci.yaml#L78-L123) (An Ngnix based image). In practice you probably want one
    or the other but not both... or neither if you are just building a theme (and not a theme + an app).  
    To enables the CI to publish on DockerHub on your behalf go to repository `Settings` tab, then `Secrets` you will need to add two new secrets:
    `DOCKERHUB_TOKEN`, you Dockerhub authorization token.  
    `DOCKERHUB_USERNAME`, Your Dockerhub username.
    We deploy the demo app at [starter.keycloakify.dev](https://starter.keycloakify.dev) using GitHub page on the branch `gh-pages` (you have to enable it).  
    To configure your own domain name please refer to [this documentation](https://docs.gitlanding.dev/using-a-custom-domain-name).
-   To release **don't create a tag manually**, the CI do it for you. Just update the `package.json`'s version field and push.
-   The `.jar` files that bundle the Keycloak theme will be attached as an asset with every GitHub release. [Example](https://github.com/InseeFrLab/keycloakify-starter/releases/tag/v0.1.0). The permalink to download the latest version is: `https://github.com/USER/PROJECT/releases/latest/download/keycloak-theme.jar`.
    For this demo repo it's [here](https://github.com/codegouvfr/keycloakify-starter/releases/latest/download/keycloak-theme.jar)
-   The CI publishes the app docker image on DockerHub. `<org>/<repo>:main` for each **commit** on `main`, `<org>/<repo>:<feature-branch-name>` for each **pull-request** on `main`
    and when **releasing a new version**: `<org>/<repo>:latest` and `<org>/<repo>:X.Y.Z`
    [See on DockerHub](https://hub.docker.com/r/codegouvfr/keycloakify-starter)

![image](https://user-images.githubusercontent.com/6702424/187989551-9461fb46-f545-4e99-b20c-e0fe4a0f773d.png)

![image](https://user-images.githubusercontent.com/6702424/187988970-99c71326-5228-4d51-8a07-dab8113387f4.png)  

If you want an example of an app that put that setup in production checkout onyxia-ui: [the repo](https://github.com/InseeFrLab/onyxia-ui), [the login](https://auth.lab.sspcloud.fr/auth/realms/sspcloud/protocol/openid-connect/auth?client_id=onyxia&redirect_uri=https%3A%2F%2Fonyxia.lab.sspcloud.fr), [the app](https://datalab.sspcloud.fr).

# Standalone vs `--external-assets`

The CI creates two jars 
- `keycloak-theme.jar`: Generated with `npx keycloakify --external-assets`, the assets, located `static/**/*`, like for example 
  `static/js/main.<hash>.js` will be downloaded from `https://starter.keycloakify.dev/static/js/main.<hash>.js` (`starter.keycloakify.dev` is 
  specified in the `package.json`.
- `standalone-keycloak-theme.jar`: Generated with `npx keycloakify`, this theme is fully standalone, all assets will be served by the 
  Keycloak server, for example `static/js/main.<hash>.js` will be downloaded from an url like `http://<your keycloak url>/resources/xxxx/login/keycloakify-starter/build/static/js/main.<hash>.js`.

More info on the `--external-assets` build option [here](https://docs.keycloakify.dev/v/v6/build-options#external-assets).  

# Docker

```bash
docker build -f Dockerfile -t codegouvfr/keycloakify-starter:test .
#OR (to reproduce how the image is built in the ci workflow):
yarn && yarn build && tar -cvf build.tar ./build && docker build -f Dockerfile.ci -t codegouvfr/keycloakify-starter:test . && rm build.tar

docker run -it -dp 8083:80 codegouvfr/keycloakify-starter:test
```

# Standalone keycloak theme

If you are only looking to create a keycloak theme, you can run theses few commands
after clicking ![image](https://user-images.githubusercontent.com/6702424/98155461-92395e80-1ed6-11eb-93b2-98c64453043f.png) to refactor the template 
and remove unnecessary files.  

```bash
rm -r src/App
mv src/keycloak-theme/* src/
rm -r src/keycloak-theme

cat << EOF > src/index.tsx
import { createRoot } from "react-dom/client";
import { StrictMode } from "react";
import { kcContext } from "./kcContext";
import KcApp from "./KcApp";

if( kcContext === undefined ){
    throw new Error(
        "This app is a Keycloak theme" +
        "It isn't meant to be deployed outside of Keycloak"
    );
}

createRoot(document.getElementById("root")!).render(
    <StrictMode>
        <KcApp kcContext={kcContext} />
    </StrictMode>
);
EOF

rm .dockerignore Dockerfile Dockerfile.ci nginx.conf

cat << EOF > .github/workflows/ci.yaml
name: ci
on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:

  build:
    runs-on: ubuntu-latest
    if: github.event.head_commit.author.name != 'actions'
    steps:
    - uses: actions/checkout@v2
    - uses: actions/setup-node@v2.1.3
      with:
        node-version: '16'
    - uses: bahmutov/npm-install@v1
    - run: yarn build
    - run: npx keycloakify
    - uses: actions/upload-artifact@v2
      with:
        name: standalone_keycloak_theme
        path: build_keycloak/target/*keycloak-theme*.jar
    - uses: actions/upload-artifact@v2
      with:
        name: build
        path: build

  check_if_version_upgraded:
    name: Check if version upgrade
    runs-on: ubuntu-latest
    needs: build
    outputs:
      from_version: \${{ steps.step1.outputs.from_version }}
      to_version: \${{ steps.step1.outputs.to_version }}
      is_upgraded_version: \${{ steps.step1.outputs.is_upgraded_version }}
    steps:
    - uses: garronej/ts-ci@v1.1.7
      id: step1
      with: 
        action_name: is_package_json_version_upgraded

  create_github_release:
    runs-on: ubuntu-latest
    needs: 
      - check_if_version_upgraded
    # We create a release only if the version have been upgraded and we are on a default branch
    # PR on the default branch can release beta but not real release
    if: |
      needs.check_if_version_upgraded.outputs.is_upgraded_version == 'true' &&
      (
        github.event_name == 'push' ||
        needs.check_if_version_upgraded.outputs.is_release_beta == 'true'
      )
    steps:
    - run: mkdir jars
    - uses: actions/download-artifact@v2
      with:
        name: standalone_keycloak_theme
    - run: mv *keycloak-theme*.jar jars/standalone-keycloak-theme.jar
    - uses: softprops/action-gh-release@v1
      with:
        name: Release v\${{ needs.check_if_version_upgraded.outputs.to_version }}
        tag_name: v\${{ needs.check_if_version_upgraded.outputs.to_version }}
        target_commitish: \${{ github.head_ref || github.ref }}
        generate_release_notes: true
        files: |
          jars/standalone-keycloak-theme.jar
        draft: false
        prerelease: \${{ needs.check_if_version_upgraded.outputs.is_release_beta == 'true' }}
      env:
        GITHUB_TOKEN: \${{ secrets.GITHUB_TOKEN }}
EOF
```

You can also remove `jwt-decode`, `keycloak-js` and `tsafe from your dependencies.  
