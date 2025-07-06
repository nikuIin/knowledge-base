[[docker]]

Issue с решением: https://github.com/docker/cli/issues/4401

А именно:

```bash
echo "autoload -U compinit; compinit" >> ~/.zshrc
docker completion zsh > $(brew --prefix)/share/zsh/site-functions/_docker
```