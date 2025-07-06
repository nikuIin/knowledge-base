
[[Шаблонный проект FastAPI]]

```.gitconfig
[filter "lfs"]
	smudge = git-lfs smudge -- %f
	process = git-lfs filter-process
	required = true
	clean = git-lfs clean -- %f
[user]
	name = van
	email = vexyzy@yandex.ru
[init]
    defaultBranch = main
[alias]
    lg = log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --branches
    lga = log --all --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --branches
    st = status
    b = branch -vv
    ba = branch --all -vv
    c = commit
[core]
	excludesfile = /Users/ivanaleksandrovci/.gitignore-global
```