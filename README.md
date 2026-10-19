<a id="readme-top"></a>

<!-- PROJECT SHIELDS -->
[![Contributors][contributors-shield]][contributors-url]
[![Forks][forks-shield]][forks-url]
[![Stargazers][stars-shield]][stars-url]
[![Issues][issues-shield]][issues-url]
[![MIT License][license-shield]][license-url]
[![LinkedIn][linkedin-shield]][linkedin-url]

<!-- PROJECT TITLE BLOCK -->
<br />
<div align="center">
  <a href="https://github.com/Elouahabi-Naoufal/git-commit-utility">
    <h3 align="center">git-commit-utility</h3>
  </a>

  <p align="center">
    A single-file Bash wrapper around the Git commands you run all day.
    <br />
    <a href="https://github.com/Elouahabi-Naoufal/git-commit-utility"><strong>Explore the source »</strong></a>
    <br />
    <br />
    <a href="https://github.com/Elouahabi-Naoufal/git-commit-utility/issues">Report Bug</a>
    &middot;
    <a href="https://github.com/Elouahabi-Naoufal/git-commit-utility/issues">Request Feature</a>
  </p>
</div>

<!-- TABLE OF CONTENTS -->
<details>
  <summary>Table of Contents</summary>
  <ol>
    <li>
      <a href="#about-the-project">About The Project</a>
      <ul>
        <li><a href="#built-with">Built With</a></li>
      </ul>
    </li>
    <li>
      <a href="#getting-started">Getting Started</a>
      <ul>
        <li><a href="#prerequisites">Prerequisites</a></li>
        <li><a href="#installation">Installation</a></li>
      </ul>
    </li>
    <li><a href="#usage">Usage</a></li>
    <li><a href="#roadmap">Roadmap</a></li>
    <li><a href="#contributing">Contributing</a></li>
    <li><a href="#license">License</a></li>
    <li><a href="#contact">Contact</a></li>
    <li><a href="#acknowledgments">Acknowledgments</a></li>
  </ol>
</details>

<!-- ABOUT THE PROJECT -->
## About The Project

`git-commit-utility` (the installed command is `comit`) collapses the three-command loop into one:

```sh
# instead of
git add .
git commit -m "Fix authentication bug"
git push origin main

# you type
comit "Fix authentication bug"
```

It is a single Bash script — about 1,300 lines, no dependencies beyond `git` itself, nothing to compile.
Copy it onto a machine and it works. Every subcommand maps to a real Git command, so nothing is abstracted
away: `comit -s` is `git status --short`, `comit --tree` is `git log --graph --oneline --all -15`, and so on.

What it actually does:

* **Commit shortcuts** — the default commit, work-in-progress commits, amend, fixup, and a force variant.
* **Branch shortcuts** — create, switch, merge, delete, plus `feature/NAME` scaffolding and timestamped
  `hotfix/YYYYMMDD-HHMM` branches cut from `main`.
* **Stash shortcuts** — stash with a message, pop the latest one, or `git add .` followed by a stash as a
  quick save point.
* **Recovery** — `--oops` / `--undo` soft-resets the last commit while keeping the changes staged.
* **Rewriting** — interactive squash of the last N commits, hard reset to a branch, and a fork sync against
  an `upstream` remote.
* **Repository setup** — interactive `git init` with branch and remote, clone-and-enter, and a GitHub
  flow that creates the repo's first commit and pushes it.
* **Insight** — status, log, diff, commit graph, contributor shortlog, object counts, remote list.

Safety rails that are actually in the code: `check_git_repo` refuses to run outside a working tree,
`--delete` refuses to remove the branch you are standing on, and `--clean`, `--hard-reset` and
`--reset` ask for confirmation before running.

State lives in `~/.comit/`: `config` for key/value settings and `token` for an optional GitHub PAT written
with `chmod 600`.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

### Built With

* [![Bash][Bash.com]][Bash-url]
* [![Git][Git.com]][Git-url]
* [![GNU coreutils][Coreutils.com]][Coreutils-url] — `date`, `grep`, `basename`
* [![OpenSSH][OpenSSH.com]][OpenSSH-url] — `ssh-keygen`, `ssh-agent`, `ssh-add`, used by `--ssh`

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- GETTING STARTED -->
## Getting Started

### Prerequisites

* `git` on your `PATH`.
* Bash 4+ (the script uses `[[ ... ]]`, regex matching inside `[[ ]]`, and `$(...)`). macOS ships Bash 3.2
  as `/bin/bash`; running the script through `zsh` or a newer Bash works, but `/usr/bin/env bash` will pick
  the old one if that is all you have.
* A Unix-like shell environment. On Windows, use WSL or Git Bash.

### Installation

1. Clone the repository

   ```sh
   git clone https://github.com/Elouahabi-Naoufal/git-commit-utility.git
   cd git-commit-utility
   ```

2. Install the script

   System-wide:

   ```sh
   sudo install -m 755 comit /usr/local/bin/comit
   ```

   Or, without `sudo`, into your own `PATH`:

   ```sh
   mkdir -p ~/.local/bin
   install -m 755 comit ~/.local/bin/comit
   export PATH="$HOME/.local/bin:$PATH"     # add this to ~/.bashrc or ~/.zshrc
   ```

3. Run the setup wizard once

   ```sh
   comit --setup
   ```

   It fills in `user.name` and `user.email` if they are unset, optionally stores a GitHub token, and sets
   `init.defaultBranch=main`, `pull.rebase=false` and `core.autocrlf=input`. These are **global** Git
   settings, so review them before accepting if you rely on different defaults elsewhere.

4. Confirm it is reachable

   ```sh
   comit --version
   comit --help
   ```

5. Optional: install the man page

   ```sh
   comit --install-man
   man comit
   ```

   This writes `comit.1` to `/usr/local/share/man/man1` when `sudo` works, otherwise to
   `~/.local/share/man/man1` and appends a `MANPATH` line to `~/.bashrc`.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- USAGE EXAMPLES -->
## Usage

```
comit [OPTION] [MESSAGE|TARGET]
```

### Everyday commits

```sh
comit "Fix authentication bug"      # add, commit, push to origin/<current branch>
comit --wip "Trying a different fix" # add, commit as "WIP: ...", no push
comit -a "Correct the typo"          # amend the previous commit, optionally force-push
comit --fixup a1b2c3d "Handle nulls"  # git commit --fixup=a1b2c3d
comit -p                             # git pull origin <current branch>
comit --sync                         # pull then push
```

### Branches

```sh
comit -b feature/login              # git checkout -b feature/login
comit -b                             # same, prompts for the branch name
comit -c main                        # git checkout main
comit --merge feature/login          # git merge feature/login
comit --delete feature/login         # git branch -d, after a y/N confirmation
comit --new-feature login "Add login form"   # feature/login, commit, push -u
comit --hotfix "Patch 500"           # hotfix/YYYYMMDD-HHMM cut from main
comit --release v1.2.1               # annotated tag, pushed to origin
```

### Rewriting and recovery

```sh
comit --squash 3                    # git rebase -i HEAD~3
comit --oops                         # git reset --soft HEAD~1, changes kept
comit --undo                         # same as --oops
comit --reset                        # interactive: git reset --hard <target> && git clean -fd
comit --hard-reset main              # same, with a confirmation prompt
comit --clean                        # git reset --hard HEAD && git clean -fd
comit --sync-fork                    # fetch upstream, merge upstream/main into main, push
```

`--clean`, `--reset` and `--hard-reset` all discard uncommitted work and untracked files. Nothing is
recoverable afterwards except through `git reflog` for the commits themselves.

### Stashing

```sh
comit --save "before refactor"       # git add . && git stash push -m "..."
comit --stash "WIP on the parser"    # git stash push -m "..."
comit --unstash                      # git stash pop
```

### Repository setup

```sh
comit --init                         # git init, pick a branch name, optionally attach a remote
comit --clone https://github.com/user/repo.git
comit --token                        # prompt for GitHub user/repo, create the first commit, push it
comit --ssh                          # generate ~/.ssh/id_ed25519 if missing, load it, print the public key
```

### Inspection

```sh
comit -s                             # git status --short
comit -l                             # git log --oneline
comit --diff                         # git diff
comit --tree                         # git log --graph --oneline --all -15
comit --who                          # git shortlog -sn
comit --size                         # git count-objects -vH
comit -r                             # git remote -v
```

### A note on `-n` and `-f` in v5.0

`-n`/`--no-push` and `-f`/`--force` are currently not honoured the way the help text describes. The script
runs `git add . && git commit && git push origin <branch>` unconditionally before it reaches the flag
handler, so:

* `comit -n "msg"` still pushes to the remote. To commit without pushing today, use
  `comit --wip "msg"` or run the Git commands directly.
* `comit -f "msg"` pushes once normally and then again with `--force`.

The `simple_push` and `smart_push` helpers at the bottom of the script contain the logic that was meant to
handle this, but nothing calls them. See the roadmap below.

### Safety notes on the GitHub token

`comit --token` builds the remote URL as `https://<token>@github.com/<user>/<repo>.git`, which writes the
token in cleartext into `.git/config` and into `git remote -v` output. `--save-token` stores it in
`~/.comit/token` with mode `600`, which is fine, but a token embedded in the remote URL is not. Prefer
`--ssh`, or a credential helper, for anything beyond a throwaway repository.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- ROADMAP -->
## Roadmap

Present and working in `comit` v5.0:

- [x] Default commit-and-push, WIP, amend, fixup and stash shortcuts
- [x] Branch create / checkout / merge / delete, `feature/` and timestamped `hotfix/` helpers
- [x] Interactive squash, reset, hard reset and clean, behind confirmation prompts
- [x] Fork sync against an `upstream` remote
- [x] Interactive `--init`, `--clone` and GitHub `--token` flows
- [x] SSH key generation for GitHub
- [x] Coloured `[INFO]`/`[SUCCESS]`/`[WARNING]`/`[ERROR]` output and a repository guard
- [x] Self-contained man page installable with `--install-man`

Open, and visible in the current code:

- [ ] Wire `simple_push` / `smart_push` into the commit path so `-n` really skips the push, `-f` pushes
      once, and a diverged branch triggers the merge / rebase / force prompt that `show_help` advertises.
- [ ] `-b|--branch` is documented as "create branch and commit", but it only runs `git checkout -b` and
      exits — either add the commit or correct the wording.
- [ ] `show_help` lists `-r, --reset` under Advanced Operations, while the parser binds `-r` to
      `--remote`; `--reset` is long-form only.
- [ ] `load_config` / `save_config` are defined but never called, so `~/.comit/config` is never actually
      written or read.
- [ ] macOS ships Bash 3.2, below the version the script assumes.
- [ ] No automated tests. The flag matrix is wide enough that a `bats` suite would catch regressions
      before release.

See the [open issues](https://github.com/Elouahabi-Naoufal/git-commit-utility/issues) for a full list of
proposed features and known issues.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- CONTRIBUTING -->
## Contributing

Contributions are what make the open source community such an amazing place to learn, inspire, and create.
Any contributions you make are **greatly appreciated**.

If you have a suggestion that would make this better, please fork the repo and create a pull request. You
can also simply open an issue with the tag "enhancement". Don't forget to give the project a star!

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the Branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

Keep the script dependency-free and POSIX-adjacent: it should still run on a stock macOS and a stock
Ubuntu with nothing but `git` installed. If you add an option, add it to `show_help` *and* to the man page
in `--install-man` — the two are maintained by hand and have already drifted apart.

### Top contributors:

<a href="https://github.com/Elouahabi-Naoufal/git-commit-utility/graphs/contributors">
  <img src="https://contrib.rocks/image?repo=Elouahabi-Naoufal/git-commit-utility" alt="contrib.rocks image" />
</a>

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- LICENSE -->
## License

Distributed under the MIT License. See [`LICENSE`](LICENSE) for more information.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- CONTACT -->
## Contact

Naoufal Elouahabi - [LinkedIn](https://www.linkedin.com/in/elouahabi-naoufal/) - elouahabi-naoufal1@gmail.com

Project Link: [https://github.com/Elouahabi-Naoufal/git-commit-utility](https://github.com/Elouahabi-Naoufal/git-commit-utility)

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- ACKNOWLEDGMENTS -->
## Acknowledgments

* [Git](https://git-scm.com/) — the tool this wraps
* [Pro Git](https://git-scm.com/book/en/v2) — the reference the command mapping follows
* [Img Shields](https://shields.io)
* [Choose an Open Source License](https://choosealicense.com)

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- MARKDOWN LINKS & IMAGES -->
<!-- https://www.markdownguide.org/basic-syntax/#reference-style-links -->
[contributors-shield]: https://img.shields.io/github/contributors/Elouahabi-Naoufal/git-commit-utility.svg?style=for-the-badge
[contributors-url]: https://github.com/Elouahabi-Naoufal/git-commit-utility/graphs/contributors
[forks-shield]: https://img.shields.io/github/forks/Elouahabi-Naoufal/git-commit-utility.svg?style=for-the-badge
[forks-url]: https://github.com/Elouahabi-Naoufal/git-commit-utility/network/members
[stars-shield]: https://img.shields.io/github/stars/Elouahabi-Naoufal/git-commit-utility.svg?style=for-the-badge
[stars-url]: https://github.com/Elouahabi-Naoufal/git-commit-utility/stargazers
[issues-shield]: https://img.shields.io/github/issues/Elouahabi-Naoufal/git-commit-utility.svg?style=for-the-badge
[issues-url]: https://github.com/Elouahabi-Naoufal/git-commit-utility/issues
[license-shield]: https://img.shields.io/github/license/Elouahabi-Naoufal/git-commit-utility.svg?style=for-the-badge
[license-url]: https://github.com/Elouahabi-Naoufal/git-commit-utility/blob/main/LICENSE
[linkedin-shield]: https://img.shields.io/badge/-LinkedIn-black.svg?style=for-the-badge&logo=linkedin&colorB=555
[linkedin-url]: https://www.linkedin.com/in/elouahabi-naoufal/
[Bash.com]: https://img.shields.io/badge/bash-4EAA25?style=for-the-badge&logo=gnubash&logoColor=white
[Bash-url]: https://www.gnu.org/software/bash/
[Git.com]: https://img.shields.io/badge/git-F05032?style=for-the-badge&logo=git&logoColor=white
[Git-url]: https://git-scm.com/
[Coreutils.com]: https://img.shields.io/badge/coreutils-437291?style=for-the-badge
[Coreutils-url]: https://www.gnu.org/software/coreutils/
[OpenSSH.com]: https://img.shields.io/badge/OpenSSH-40739C?style=for-the-badge&logo=openssh&logoColor=white
[OpenSSH-url]: https://www.openssh.com/
