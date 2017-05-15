# fzf For The Win!

---

# But first, an alternate definition of duck-typing

---

# Duck typing

> Si t'as une vache avec un bec qui fait coin-coin, et ben tu peux
> l'utiliser comme un canard.

---

# En images

![canard-vache](canard-vache.jpg)

---

# Plan

* Je vous montre plein de commandes que j'utilisais avant
* Puis je vous montre comment j'ai tout remplacé par `fzf`
* Avec des démos entre les deux

---

# Avant fzf

![dark times](dark-times.jpg)

---

# find

    !bash
    $ find . -name "*foo*"
    # ou
    $ find . -iname "*foo*"

---

# Naviguer dans l'historique zsh

* `CTRL-R`

* Appuyer sur la flèche du haut

        !bash
        bindkey '^[[A' history-beginning-search-backward
        bindkey '^[[B' history-beginning-search-forward

        $ ssh <fléche haut>
        # Cherche toutes les commandes commençant par `ssh`

* Quand ça veut pas

        !bash
        $ grep 'something' ~/.zshhistory
        # ou même
        $ vi ~/.zshhistory
        # et puis `:saveas my_script.sh`


---


# Lister les buffers dans vim

    !vim
    :ls
    " puis
    :buffer <number>
    " ou
    :buffer foo<TAB>


---

# Naviguer dans l'historique vim

    !vim
    :oldfiles
    " ou
    :browse oldfiles

![oldfiles](oldfiles.png)

---

# Visiter les répertoires déjà fréquentés

* autojump
* z
* ...

        !bash
        $ cd ~/a/long/path/to/foo
        # '~/a/long/path/to/foo' est inséré dans une base de donnée
        $ cd ~/an/other/path/to/bar
        # même chose pour ~/an/other/path/to/bar
        $ z foo
        # change pour le chemin de foo

---

# Cherchons le pattern

![patterns ?](patterns.jpg)

---


# Similarités

Dans tous les cas on a une liste dans laquelle on voudrait sélectionner un
résultat *et un seul*

Dans la plupart des cas, les listes sont plus ou moins triées.

Et en général on veut ne taper qu'un bout de ce qu'on cherche, et pas s'embêter
avec la casse.

---

# Problèmes

Chacun des outils présenté a sa propre façon de faire.

Et en plus, chacun a des côtés pénibles

---

# find

Extrait de mon historique zsh:

    !bash
    $ find -name *foo*
    # Oups, faut mettre un point avant
    $ find . -name *foo*
    # Oups, faut échapper les étoiles:
    $ find . -name "*foo*"
    # Oups, je voulais `-iname` en pas `-name`
    $ find . -iname "*foo*"

---

# L'historique zsh

Jamais réussi à le faire marcher sur plusieurs distros en même temps ...

Une fois qu'on a tapé `CTRL-R` on ne voit qu'un choix à la fois, et c'est
pas pratique.


---


# `:list`, `:buffer`

`:list` affiche des infos utiles
(par example, le fichier 'alternate', qu'on peut éditer avec
`:b#`)

`:buffer` et la complétion marchent bien, sauf quand on bosse avec plein de
`index.js` dans plein de répertoires différents ...

---

# `:oldfiles`

`:oldfiles` sert pas à grand chose en soi.

Il faut en plus se taper le "Press enter to continue, q to quit ..."

Et `:browse oldfiles` c'est long à taper.

Pas de recherche possible no plus :/

---

# `z`, `autojump`

L'heuristique essaye de deviner le répertoire dans lequel vous voulez aller.

Mais quand c'est ambigu ça marche pas bien.

(Et c'est très frustrant)

    !bash
    z foo
    # /path/to/foo
    z foobar
    # /path/to/foobar

---

# fzf

---

# Fait un truc et le fait bien

    !bash
    cat myliste.txt | fzf

Affiche tout le contenu de `maliste.txt`, et permet de sélectionner
rapidement le bon choix.

Pas besoin de taper *exactement* un bout de ligne: c'est ça la partie
"fuzzy"

---

# Installation

    !bash
    git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf
    ~/.fzf/install

C'est du `Go` donc:

* Ça marche partout
* Ça a pas de dépendances
* C'est **rapide**

---

# Démo 1

Fonctionnement basique de `fzf`

---

# En asynchrone c'est mieux

Quand la liste est très grande, `fzf` permet quand même de sélectionner
*tout en continuant à alimenter la liste originale*

---

# Démo 2

Lister les fichiers d'un répertoire:

`CTRL-T` par défaut

---

# Remplacer find?

Quand on veut éxecuter *une* commande sur *un fichier* particulier et qu'on
connaît à peu près son nom, `fzf` est très bien.

Pour lancer une commande avec plusieurs fichiers, on peut faire:

    !bash
    $ ma-commande */**.txt

Et si on veut lancer `ma-commande` une fois pour chaque fichier:

    ! bash
    $ find . -name *.txt | xargs -n1 ma-commande


(C'est l'option `-n` de `xargs`, peu connue)

---

# Dans vim

`fzf.vim` est un plug-in qui enrichit le plug-in vim installé par défaut avec `fzf`

Il fournit des commandes "clé en main" comme `:Buffers`, `:History`, ou
`:GitFiles`.

Après installation, il suffit de configurer vos raccourcis:


    !vim
    nnoremap <leader>p :History<CR>
    nnoremap <leader>b :Buffers<CR>
    nnoremap <leader>t :Files<CR>

---

# fzf.vim

Permet aussi de créer vos propres commandes avec un peu de `vimscript`:


    !vim

    function SelectSomething()
    call fzf#run({
        \ 'source': "my-cmd",
        \ 'sink': ":DoSomething"
        \})
    endfunction

    command! -nargs=0 SelectSomething :call SelectSomething()


Ici, on va lancer `my-cmd` pour alimenter la liste `fzf`, puis
passer le résultat à `:DoSomething`

---

# Remplacer autojump


Une <del>pile de hacks</del> collection de petit bouts de code et réglages


---

# Stocker une liste de chemins

Dans `~/.local/share/zsh/cwd.json`:

    !json
    {
      "/path/to/foo": 2,
      "/path/to/foo/src": 3,
      "/path/to/bar": 1
    }

---


# Un petit bout de Python

Un petit bout de Python/argparse pour implémenter:

* `cwd add CHEMIN`: Incrémente le compteur pour le chemin donné
* `cwd edit`
* `cwd clean` (Supprime les répertoires qui n'existent plus)
* `cwd list` Pour lister les chemins dans l'ordre

<br />

<small>Vous avez le droit d'utiliser des langages/libraries moins bien si vous
préférez</small>

---

# Se hooker dans zsh

Pas compliqué:

    !bash
    function register_cwd() {
        cwd-history add "$(pwd)"
    }
    typeset -gaU register_cwd
    chpwd_functions+=register_cwd


Ici `register_cwd` sera appelée à chaque fois que `zsh` change de répertoire
courant.

---

# Définir un "widget"

Dans `.zshrc`:

    !bash
    fzf-cd-widget() {
    ret=$(cwd-history list | fzf --tac)
    builtin cd "${ret}"
    if [[ $? -ne 0 ]]; then
        cwd-history remove "${ret}"
    fi
    }
    zle -N fzf-cd-widget

---

# Assigner un racourci:

Dans `.zshrc`:

    !bash
    bindkey '\ed' fzf-cd-widget
    bindkey -s '\ec' '\ed\n'

Oui il y en a deux. Le second appelle le premier et rajoute '\n' à la fin.
J'ai pas trouvé mieux :/

---

# Un dernier truc cool

On peut aussi partager des infos entre vim et le shell.

---

# Changer de répertoire neovim quitte

    !vim
    " Write cwd when leaving
    function! WriteCWD()
    call writefile([getcwd()], "/tmp/nvim-cwd")
    endfunction

    autocmd VimLeave * silent call WriteCWD()


    !bash
    " Change working dir after neovim exits
    function vim {
      neovim $*
      if [[ $? -eq 0 ]]; then
          cd "$(cat /tmp/nvim-cwd 2>/dev/null || echo .)"
      fi
    }


Oui, c'est dégueu

---

# Lire les chemins dans vim

Plus propre:

    !vim
    function ListWorkingDirs()
    call fzf#run({
            \ 'source': "cwd-history list",
            \ 'sink': "cd"
            \})
    endfunction

    command! -nargs=0 ListWorkingDirs :call ListWorkingDirs()
    nnoremap <leader>l :ListWorkingDirs<CR>

---

# Une démo pour finir