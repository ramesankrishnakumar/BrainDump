# Local Config

## HomeBrew

- raycast - spotlight replacement <br>
  > enable homebrew extension in raycast and install brew packages via raycast OR <br>
  > add all brew package names to a file separated by newline AND <br>
  > xargs brew install < 'apps.txt' <br>

```console
brew install --cask raycast
```

- Dbeaver

```console
brew install --cask dbeaver-community
```

- Iterm2

```console
brew install --cask iterm2
```

- PyEnv

```console
brew install pyenv
```

- awscli

```console
brew install awscli
```

- kustomize

```console
brew install kustomize
```

- podman

```console
brew install podman
```

- psql command line utility

```console
brew install libpq
```

- json processor

```console
brew install jq
```

## Poetry - Python

```
curl -sSL https://install.python-poetry.org | python3 -
poetry config virtualenvs.create true
poetry config virtualenvs.in-project true
```

## ITerm

- [Install ohmyzsh](https://ohmyz.sh/#install)
- [Install p10k](https://github.com/romkatv/powerlevel10k?tab=readme-ov-file#getting-started)
- [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting/blob/master/INSTALL.md#oh-my-zsh)
- [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions/blob/master/INSTALL.md#oh-my-zsh)
- Add the below lines to ~/.zshrc <br>
  > export JAVA_HOME=`/usr/libexec/java_home -v 11` <br>
  > plugins=( git zsh-syntax-highlighting zsh-autosuggestions ) <br>

## Git SSH keys

- [create](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
- [add](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)
- [verify](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/testing-your-ssh-connection)

## Git mutiple SSH config

```
Host github.abc.com
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile /Users/kramesan/.ssh/id_rsa

Host github.com
  AddKeysToAgent yes
  UseKeyChain yes
  IdentityFile ~/.ssh/github_com
```

## nvm

- [refer](https://github.com/nvm-sh/nvm?tab=readme-ov-file#installing-and-updating) - you'll end up doing something like below

```
# installs nvm (Node Version Manager)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash

# download and install Node.js (you may need to restart the terminal)
nvm install 22

# verifies the right Node.js version is in the environment
node -v # should print `v22.11.0`

# verifies the right npm version is in the environment
npm -v # should print `10.9.0`
```

## Chrome

- [Simple Super Highlighter](https://chromewebstore.google.com/detail/super-simple-highlighter/hhlhjgianpocpoppaiihmlpgcoehlhio)
- [Keepa](https://chromewebstore.google.com/detail/keepa-amazon-price-tracke/neebplgakaahbhdphmkckjjcegoiijjo)
- [Video Speed Controller](https://chromewebstore.google.com/detail/video-speed-controller/nffaoalbilbmmfgbnbgppjihopabppdk)
- [Tabliss](https://chromewebstore.google.com/detail/tabliss-a-beautiful-new-t/hipekcciheckooncpjeljhnekcoolahp)
- [UV Weather](https://chromewebstore.google.com/detail/uv-weather/ngeokhpbgoadbpdpnplcminbjhdecjeb)
- [ZED: Zoom Easy Downloader] (https://chromewebstore.google.com/detail/zed-zoom-easy-downloader/pdadlkbckhinonakkfkdaadceojbekep)

## VS Code

- add this to settings.json cmd + shift + p -> settings json
<details> <summary>Settings.json</summary>

```json
{
  "redhat.telemetry.enabled": false,
  "editor.fontSize": 20,
  "RainbowBrackets.depreciation-notice": false,
  "files.autoSave": "onFocusChange",
  "editor.wordWrap": "on",
  "workbench.editor.autoLockGroups": {
    "decompiled.javaClass": true
  },
  "workbench.colorTheme": "Cobalt2",
  "workbench.iconTheme": "vscode-icons",
  "security.workspace.trust.untrustedFiles": "open",
  "editor.accessibilitySupport": "off",
  "security.promptForLocalFileProtocolHandling": false,
  "editor.formatOnSave": true,
  "eslint.codeActionsOnSave.rules": null,
  "editor.linkedEditing": true,
  "editor.minimap.sectionHeaderFontSize": 12,
  "terminal.integrated.env.linux": {},
  "terminal.integrated.fontSize": 20,
  "javascript.updateImportsOnFileMove.enabled": "always",
  "explorer.confirmDelete": false,
  "editor.fontWeight": "normal",
  "workbench.sideBar.location": "right",
  "tailwindCSS.experimental.classRegex": [],
  "files.associations": {
    "*.css": "tailwindcss"
  },
  "editor.quickSuggestions": {
    "strings": "on"
  },
  "explorer.confirmPasteNative": false,
  "telemetry.telemetryLevel": "off",
  "explorer.confirmDragAndDrop": false,
  "update.showReleaseNotes": false,
  "extensions.ignoreRecommendations": true,
  "codium.codeCompletion.enable": false,
  "github.copilot.enable": {
    "*": true,
    "plaintext": false,
    "markdown": false,
    "scminput": false
  },
  "editor.stickyScroll.enabled": false,
  "[css]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[handlebars]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[html]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[javascriptreact]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[json]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[jsonc]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[markdown]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "diffEditor.ignoreTrimWhitespace": false
  },
  "[scss]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[typescriptreact]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "editor.minimap.enabled": false,
  "fontshortcuts.defaultFontSize": 20,
  "fontshortcuts.defaultTerminalFontSize": 20,
  "fontshortcuts.step": 1,
  "cSpell.enabled": true,
  "editor.lineHeight": 0,
  "terminal.integrated.fontFamily": "MesloLGS NF",
  "cSpell.userWords": ["Doordash"],
  "workbench.editor.enablePreview": false
}
```

</details>

- keyboard bindings

<details><summary>keybindings.json</summary>
  
``` json
[
  {
    "key": "cmd+s",
    "command": "-workbench.action.files.saveAll"
  },
  {
    "key": "cmd+n",
    "command": "-editor.action.sourceAction",
    "when": "editorHasCodeActionsProvider && editorTextFocus && !editorReadonly"
  },
  {
    "key": "cmd+1",
    "command": "-workbench.action.focusFirstEditorGroup"
  },
  {
    "key": "cmd+b",
    "command": "workbench.action.toggleSidebarVisibility"
  },
  {
    "key": "cmd+b",
    "command": "-workbench.action.toggleSidebarVisibility"
  },
  {
    "key": "cmd+1",
    "command": "workbench.view.explorer",
    "when": "viewContainer.workbench.view.explorer.enabled"
  },
  {
    "key": "shift+cmd+e",
    "command": "-workbench.view.explorer",
    "when": "viewContainer.workbench.view.explorer.enabled"
  },
  {
    "key": "shift+cmd+e",
    "command": "workbench.view.explorer",
    "when": "viewContainer.workbench.view.explorer.enabled"
  },
  {
    "key": "cmd+s",
    "command": "saveAll"
  },
  {
    "key": "alt+cmd+s",
    "command": "-saveAll"
  },
  {
    "key": "shift+cmd+/",
    "command": "editor.action.blockComment",
    "when": "editorTextFocus && !editorReadonly"
  },
  {
    "key": "shift+alt+a",
    "command": "-editor.action.blockComment",
    "when": "editorTextFocus && !editorReadonly"
  },
  {
    "key": "shift+alt+a",
    "command": "editor.action.blockComment",
    "when": "editorTextFocus && !editorReadonly"
  }
]

```
</details>

## set JAVA_HOME
```

export JAVA_HOME=`/usr/libexec/java_home -v 11`

```

```
