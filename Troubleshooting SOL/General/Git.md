Personal-access tokens: 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
Store so it wont ask again
	

VSCode is SUPERIOR for Merge Conflicts

Changes are stored as local changes, carries to other branches when you move (if)
Stash is not unique to any one branch, but unique to the entire repo

Commands:
	Clone particular branch
		git clone <link> --branch <branch-name>
	Init local git repo from remote
		git clone "[repo link]"
	Standard code change process:
		git add --all or [specific filepath]
		git commit -m "[commit message]"
		git push
	Track branches:
		git branch -a -> gives you all branches on remote repo
		git branch -> gives you locally tracked branches on remote repo
	Change git editor:
		git config --global core.editor nano
	Edit commit:
		git commit --amend --author="Author Name <email>"
	Temporary work without altering main branch:
		git stash
			git stash list
			git stash drop <id>
			git stash apply -> leaves in stash
			git stash pop -> pops from stash
	Reset Commit:
		git reset --soft HEAD~1 -> leaves changes as local changes
		Reset ^:
			git reset HEAD@{1}
	Delete remote branch:
		git push -d origin <branch-name>
	Delete local branch
		git branch -d <branch-name>
	Checkout remote branch:
		Make sure origin is a remote
		run git fetch to download latest
		git checkout -b <new-name> <origin/old-name>	
	Unstage:
		git restore --staged --all
	Set upstream:
		git branch --set-upstream origin <branch>
	Change remote:
		git remote set-url origin <remote url>
	Add remote for first time and checkout branch:
		git remote add origin <remote url> or set-url origin
		verify
			git remote get-url origin
		git fetch
		git branch -a
		git stash
		git checkout -b <different name> origin/main
		git stash pop
		git branch -d <branch from other repo>
		git branch -m <new name>
	Check git repo info:
		git config --list
	Undo recent remote commit:
		git reset HEAD^
		git push origin +HEAD	
	Open repository without clone:
		https://ivan.bessarabov.com/blog/cloning-git-repo-withou-git-clone
	Recursive clone of repo
		git clone --recursive
		
	
Problems:
	If untracked working tree error
		NUKE THE REPO and start over
	

