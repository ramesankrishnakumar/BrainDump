# env-config
# GIT config
$ git config --global user.name "John Doe"
$ git config --global user.email johndoe@example.com
$ git config --global fetch.prune true

# SDK man config
$ sdk install java 11-corretto /Library/Java/JavaVirtualMachines/amazon-corretto-11.jdk/Contents/Home/
$ sdk install java 11-corretto /Library/Java/JavaVirtualMachines/amazon-corretto-11.jdk/Contents/Home/
$ sdk install java 17-corretto /Library/Java/JavaVirtualMachines/amazon-corretto-17.jdk/Contents/Home/
$ sdk install java 21-corretto /Library/Java/JavaVirtualMachines/amazon-corretto-21.jdk/Contents/Home/
$ sdk default java 11-corretto

# HomeBrew 
brew install --cask dbeaver-community
brew install awscli

#Jenv
$ brew install jenv
$ echo 'export PATH="$HOME/.jenv/bin:$PATH"' >> ~/.zshrc
$ echo 'eval "$(jenv init -)"' >> ~/.zshrc

$ jenv add /Library/Java/JavaVirtualMachines/amazon-corretto-11.jdk/Contents/Home
$ jenv add /Library/Java/JavaVirtualMachines/amazon-corretto-17.jdk/Contents/Home
$ jenv add /Library/Java/JavaVirtualMachines/amazon-corretto-21.jdk/Contents/Home/
$ jenv enable-plugin maven
$ jenv enable-plugin gradle
$ jenv enable-plugin export
