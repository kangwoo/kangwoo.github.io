---
title:  "macOS 개발 환경 설절하기"
classes: wide
date: 2019-09-14T09:21:10+09:00
categories: [devops, macOS]
tags: [macOS]
---


## 필수 프로그램
### Xcode
- 설치
    ```bash
    xcode-select --install
    ```

### homebrew
- 설치
    ```bash
    /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
    ```
- 기타
    - [homebrew 홈페이지](https://brew.sh/)


### git
- 설치
    ```bash
    brew install git git-lfs
    ```

- 설정
    ```bash
    git config --global user.name "Your name"
    git config --global user.email "you@your-domain.com"
    git config --global core.precomposeunicode true
    git config --global core.quotepath false
    ```

### iTerm2
- 설치
    ```bash
    brew cask install iterm2
    ```

### zsh with oh-my-zsh
- 설치
```bash
brew install zsh zsh-completions

# 폴트 설치
git clone https://github.com/powerline/fonts.git
cd fonts
./insta..sh

sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```


### fzf
- 설치
```bash
brew install fzf

# To install useful key bindings and fuzzy completion:
$(brew --prefix)/opt/fzf/install
```


```bash
echo 'export PATH="/usr/local/opt/gettext/bin:$PATH"' >> ~/.zshrc
```


### jq
https://stedolan.github.io/jq/

### asciinema
https://asciinema.org/

### ngrok
