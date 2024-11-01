# Local Config
## HomeBrew 
- raycast - spotlight replacement <br>
> enable homebrew extension in raycast and install brew packages via raycast  OR <br>
> add all brew package names to a file separated by newline AND <br>
> xargs brew install < 'apps.txt' <br>

``` console
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
``` console
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
```console
brew install powerlevel10k
```
``` console
echo "source $(brew --prefix)/share/powerlevel10k/powerlevel10k.zsh-theme" >>~/.zshrc <br>
```
``` console
brew install zsh-syntax-highlighting zsh-autosuggestions
```
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

## VS Code 
- add this to settings.json  cmd + shift + p -> settings json
```
"editor.stickyScroll.enabled": false,
# disable the minimap on the right
"editor.minimap.enabled": false,
# tag editing good for html
"editor.linkedEditing": true,
```

## set JAVA_HOME
```
export JAVA_HOME=`/usr/libexec/java_home -v 11`
```
