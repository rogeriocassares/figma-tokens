# figma-tokens

Create a node project
```bash
npm init -y
```

Install style-dictionary
```bash
npm i style-dictionary
```

Execute `style-dictionary` basic
```bash
npx style-dictionary init basic
```

Modify `package.json` to build `style-dictionary`
```bash
...
"scripts": {
    "build": "style-dictionary build"
  },
...
```

In Figma, click in `Plugins` -> `Design Token` -> `Export Design Token File` -> `Save & Export`.

Then, rename as `token.js` and insert as token.json in `/input/token.json` 
```bash
{
  "color": {
    "white": {
      "description": "",
      "type": "color",
      "value": "#ffffffff",
      "blendMode": "normal",
      "extensions": {
        "org.lukasoppermann.figmaDesignTokens": {
          "styleId": "S:4e7c9cf8df57bdd05846024940cd0af4cb141436,",
          "exportKey": "color"
        }
      }
    },
    "purple": {
      "500": {
        "description": "",
        "type": "color",
        "value": "#8257e6ff",
        "blendMode": "normal",
        "extensions": {
          "org.lukasoppermann.figmaDesignTokens": {
            "styleId": "S:c87e728900e6f6e7ca21f6b05dc941252f0d1162,",
            "exportKey": "color"
          }
        }
      }
    },
    "gray": {
      "200": {
        "description": "",
        "type": "color",
        "value": "#c1c1c4ff",
        "blendMode": "normal",
        "extensions": {
          "org.lukasoppermann.figmaDesignTokens": {
            "styleId": "S:877ce7d25f2350925195cba43eb4c4bea457fa46,",
            "exportKey": "color"
          }
        }
      },
      "500": {
        "description": "",
        "type": "color",
        "value": "#50505cff",
        "blendMode": "normal",
        "extensions": {
          "org.lukasoppermann.figmaDesignTokens": {
            "styleId": "S:e3ba9ad5240451b441c5e9ce4617e5cea5bee249,",
            "exportKey": "color"
          }
        }
      },
      "800": {
        "description": "",
        "type": "color",
        "value": "#1c1c1fff",
        "blendMode": "normal",
        "extensions": {
          "org.lukasoppermann.figmaDesignTokens": {
            "styleId": "S:8df1f27acc69b9dfa629c364632f5c6ca07d6634,",
            "exportKey": "color"
          }
        }
      },
      "900": {
        "description": "",
        "type": "color",
        "value": "#121214ff",
        "blendMode": "normal",
        "extensions": {
          "org.lukasoppermann.figmaDesignTokens": {
            "styleId": "S:d2fa0f28094afe9f8274071589ab22306c90e89b,",
            "exportKey": "color"
          }
        }
      }
    }
  },
  "effect": {
    "shadow": {
      "md": {
        "description": null,
        "type": "custom-shadow",
        "value": {
          "shadowType": "dropShadow",
          "radius": 4,
          "color": "#ffffff1a",
          "offsetX": 0,
          "offsetY": 2,
          "spread": 0
        },
        "extensions": {
          "org.lukasoppermann.figmaDesignTokens": {
            "styleId": "S:f887c123386839ef6380cafd527642edc90814e2,",
            "exportKey": "effect"
          }
        }
      }
    }
  }
}
```
In `config.json`, modify the format to achieve the desired format:
```bash
{
  "source": ["input/**/*.json"],
  "platforms": {
    "css": {
      "transformGroup": "css",
      "buildPath": "build/css/",
      "files": [{
        "destination": "variables.css",
        "format": "css/variables"
      }]
    },
    "js": {
      "transformGroup": "js",
      "buildPath": "build/js/",
      "files": [{
        "destination": "tokens.js",
        "format": "javascript/module-flat"
      }]
    }
  }
}
```

Create a `.gitignore` file containing:
```bash
node_modules
input/*.json
```

Create a `.gitkeep` file inside `/input`
```bash
touch ./input/.gitkeep
```
Create a github workflow to achieve Github Action in dispatched event from remote uri
```yaml
mkdir -p .github/workflows/
touch .github/workflows/update-tokens.yaml
```
In `.github/workflows/update-tokens.yaml`:
```bash
name: Generate design tokens

on:
  repository_dispatch:
    types: update-tokens

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Create input JSON
        id: create-json
        uses: jsdaniell/create-json@1.1.2
        with:
          name: ${{ github.event.client_payload.filename }}
          json: ${{ github.event.client_payload.tokens }}
          dir: 'input'

      - name: Install Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 16

      - name: Install dependencies
        run: npm ci
      
      - name: Build
        run: npm run build

      - name: Create PR 
        uses: peter-evans/create-pull-request@v4
        with:
          commit-message: "style: Update design tokens"
          title: ${{ github.event.client_payload.commitMessage || 'Update design tokens' }}
          body: "Design tokens have been updated via Figma and need to be reviewed."
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          BRANCH_NAME: 'main'
```

In Figma, go to `Plugins` -> `Design Token` -> `Send Design Token to Url`. Then configure as follows:
* Event type: `update-tokens`
* Server url: `https://api.github.com/repos/rogeriocassares/figma-tokens/dispatches`
* Accept header: `application/vnd.github.everest-preview+json`
* Content-Type header: `text/plain;charset=UTF-8`
* Auth type: `(Github) token`
* Access token: `GITHUB_TOKEN_HERE`
* Commit Message: `Update Token`

Then `Save & Export` and go to `Github repository` -> `Actions`. See if the PR is automatically generated, the merge it to the `main` branch.

# TODO
github action to publish the Merged PR to npm
