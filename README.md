# percol

                                    __
        ____  ___  ______________  / /
       / __ \/ _ \/ ___/ ___/ __ \/ /
      / /_/ /  __/ /  / /__/ /_/ / /
     / .___/\___/_/   \___/\____/_/
    /_/

percol adds flavor of interactive selection to the traditional pipe concept on UNIX

## Installation

First, clone percol repository and go into the directory.

    $ git clone git://github.com/mooz/percol.git
    $ cd percol

Then, run a command below.

    $ sudo python setup.py install

If you don't have a root permission (or don't wanna install percol with sudo), try next one.

    $ python setup.py install --prefix=~/.local
    $ export PATH=~/.local/bin:$PATH

## Usage

Specifying a filename.

    $ percol /var/log/syslog

Specifying a redirecition.

    $ ps aux | percol


## Matching Method

By default, percol interprets input queries by users as **string**. If you prefer **regular expression**, try `--match-method` command line option.

    $ percol --match-method regex

### Migemo support

From version 0.0.2, percol supports **migemo** (http://0xcc.net/migemo/) for `--match-method` experimentally.

    $ percol --match-method migemo

This feature requires following external modules for now.

- C/Migemo (http://code.google.com/p/cmigemo/)
- PyMigemo (http://www.atzm.org/etc/pymigemo.html)

#### Dictionary settings

By default, percol assumes the path of a dictionary for migemo is `/usr/local/share/migemo/utf-8/migemo-dict`. If the dictionary is located in a different place, you should tell the location via `rc.py`.

For example, if the path of the dictionary is `/path/to/a/migemo-dict`, put lines below into your `rc.py`.

    from percol.finder import FinderMultiQueryMigemo
    FinderMultiQueryMigemo.dictionary_path = "/path/to/a/migemo-dict"

#### Minimum query length

If the query length is **too short**, migemo generates **very long** regular expression. To deal with this problem, percol does not pass a query if the length of the query is shorter than **2** and treat the query as raw regular expression.

To change this behavior, change the value of `FinderMultiQueryMigemo.minimum_query_length` like following settings.

    from percol.finder import FinderMultiQueryMigemo
    FinderMultiQueryMigemo.minimum_query_length = 1

## Tips

### Selecting multiple candidates

You can select and let percol to output multiple candidates by `percol.command.toggle_mark_and_next()` (which is bound to `C-SPC` by default).

`percol.command.mark_all()`, `percol.command.unmark_all()` and `percol.command.toggle_mark_all()` are useful to mark / unmark all candidates at once.

## Example

### Interactive pgrep / pkill

Here is an interactive version of pgrep,

    $ ps aux | percol | awk '{ print $2 }'

and here is an interactive version of pkill.

    $ ps aux | percol | awk '{ print $2 }' | xargs kill

For zsh users, command versions are here (`ppkill` accepts options like `-9`).

    function ppgrep() {
        if [[ $1 == "" ]]; then
            PERCOL=percol
        else
            PERCOL="percol --query $1"
        fi
        ps aux | eval $PERCOL | awk '{ print $2 }'
    }
    
    function ppkill() {
        if [[ $1 =~ "^-" ]]; then
            QUERY=""            # options only
        else
            QUERY=$1            # with a query
            shift
        fi
        ppgrep $QUERY | xargs kill $*
    }

### zsh history search

In your `.zshrc`, put the lines below.

    function exists { which $1 &> /dev/null }
    
    if exists percol; then
        function percol_select_history() {
            local tac
            exists gtac && tac=gtac || tac=tac
            BUFFER=$($tac $HISTFILE | sed 's/^: [0-9]*:[0-9]*;//' | percol --query "$LBUFFER")
            CURSOR=$#BUFFER         # move cursor
            zle -R -c               # refresh
        }
    
        zle -N percol_select_history
        bindkey '^R' percol_select_history
    fi

Then, you can display and search your zsh histories incrementally by pressing `Ctrl + r` key.

### tmux

Here are some examples of tmux and percol integration.

    bind b split-window "tmux lsw | percol --initial-index $(tmux lsw | awk '/active.$/ {print NR-1}') | cut -d':' -f 1 | xargs tmux select-window -t"
    bind B split-window "tmux ls | percol --initial-index $(tmux ls | awk '/attached.$/ {print NR-1}') | cut -d':' -f 1 | xargs tmux switch-client -t"

By putting above 2 settings into `tmux.conf`, you can select a tmux window with `${TMUX_PREFIX} b` keys and session with `${TMUX_PREFIX} B` keys.

## Configuration

Configuration file for percol should be placed under `${HOME}/.percol.d/` and named `rc.py`.

Here is an example `~/.percol.d/rc.py`.

    # X / _ / X
    percol.view.PROMPT  = ur"<bold><yellow>X / _ / X</yellow></bold> %q"
    
    # Emacs like
    percol.import_keymap({
        "C-h" : lambda percol: percol.command.delete_backward_char(),
        "C-d" : lambda percol: percol.command.delete_forward_char(),
        "C-k" : lambda percol: percol.command.kill_end_of_line(),
        "C-y" : lambda percol: percol.command.yank(),
        "C-a" : lambda percol: percol.command.beginning_of_line(),
        "C-e" : lambda percol: percol.command.end_of_line(),
        "C-b" : lambda percol: percol.command.backward_char(),
        "C-f" : lambda percol: percol.command.forward_char(),
        "C-n" : lambda percol: percol.command.select_next(),
        "C-p" : lambda percol: percol.command.select_previous(),
        "C-v" : lambda percol: percol.command.select_next_page(),
        "M-v" : lambda percol: percol.command.select_previous_page(),
        "M-<" : lambda percol: percol.command.select_top(),
        "M->" : lambda percol: percol.command.select_bottom(),
        "C-m" : lambda percol: percol.finish(),
        "C-j" : lambda percol: percol.finish(),
        "C-g" : lambda percol: percol.cancel(),
    })
