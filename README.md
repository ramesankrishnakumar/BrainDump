# env-config
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
- node js
```console
brew install node
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



## Chrome
- [Simple Super Highlighter](https://chromewebstore.google.com/detail/super-simple-highlighter/hhlhjgianpocpoppaiihmlpgcoehlhio)
- [Keepa](https://chromewebstore.google.com/detail/keepa-amazon-price-tracke/neebplgakaahbhdphmkckjjcegoiijjo)
- [Video Speed Controller](https://chromewebstore.google.com/detail/video-speed-controller/nffaoalbilbmmfgbnbgppjihopabppdk)
- [Tabliss](https://chromewebstore.google.com/detail/tabliss-a-beautiful-new-t/hipekcciheckooncpjeljhnekcoolahp)
- [UV Weather](https://chromewebstore.google.com/detail/uv-weather/ngeokhpbgoadbpdpnplcminbjhdecjeb)

## set JAVA_HOME
```
export JAVA_HOME=`/usr/libexec/java_home -v 11`
```
