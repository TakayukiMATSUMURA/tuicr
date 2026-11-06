# Keybindings

Full reference. Press `?` inside tuicr for an in-app version of this list.

Keys use Emacs notation: `C-n` is Ctrl+n, `M-x` is Meta (Option/Alt)+x, and `C-SPC`
is Ctrl+Space. `M-` keys need the terminal to send Option/Alt as Meta. `C-g` works
wherever `Esc` does.

`<leader>` defaults to `;`. Override it with `leader = ","` in `~/.config/tuicr/config.toml`.

## Navigation

| Key | Action |
|-----|--------|
| `C-n` / `↓` | Scroll down |
| `C-p` / `↑` | Scroll up |
| `C-b` / `←` | Scroll left |
| `C-f` / `→` | Scroll right |
| `C-v` / `M-v` | Page down / up (also `PageDown` / `PageUp`) |
| `M-<` / `M->` | Go to first / last file |
| `{N}M->` | Go to source line N in current file |
| `{N}{motion}` | Count prefix — repeats `C-n` / `C-p` / `C-b` / `C-f` / `{` / `}` / `[` / `]` `N` times |
| `{` / `}` | Jump to previous / next file |
| `[` / `]` | Jump to previous / next hunk |
| `m` / `M` | Jump to next / previous comment |
| `C-s` | Search within diff (case-insensitive regex); matches on diff content are highlighted and the status bar shows the `[current/total]` position (headers, comments, and PR info are searchable but not highlighted) |
| `C-s` / `C-r` | Next / previous search match while matches are highlighted (wraps around) |
| `C-g` / `Esc` | Clear search-match highlighting; the next `C-s` starts a new search |
| `Enter` | Expand or collapse hidden context between hunks |
| `zt` | Scroll cursor to top of screen |
| `zz` | Center cursor on screen |
| `zb` | Scroll cursor to bottom of screen |
| `Ctrl-l` | Clear and redraw the whole screen |

`Ctrl-l` works in every mode, including while typing a comment, a `:` command or
a search. Use it when the terminal leaves stale text on screen, for example after
a font size change in a window that wasn't visible. tuicr also redraws in full
whenever the terminal reports a resize.

## Diff search

`C-s` opens the search prompt. Type a pattern and press `Enter` or `C-s` to jump
to the first match after the cursor. While matches are highlighted, `C-s` and `C-r`
step to the next and previous match. `C-g` clears the highlight, so the next `C-s`
opens the prompt again. With nothing highlighted, `C-r` opens the same prompt and
searches forward.

Diff search uses the Rust `regex` syntax, case-insensitively by default,
like the file-tree include/exclude filters. For example, `foo|bar` matches either
word, `^fn\s+` finds function declarations, and `foo\.bar` matches a literal dot.
Plain-text searches keep their existing Unicode case folding. The same compiled
pattern drives navigation and highlighting in both diff layouts; side-by-side
columns are matched independently. Zero-width matches (such as `^`) are navigable
but have no characters to highlight. An invalid expression reports an error
without moving the cursor or replacing the previous valid search. File-tree and
help searches remain text searches.

## Help

Press `?` to open help.

| Key | Action |
|-----|--------|
| `C-s` | Search within help (case-insensitive); next match once a search has matched |
| `C-r` | Previous help search match |
| `C-n` / `C-p` | Scroll down / up |
| `q` / `?` / `C-g` / `Esc` | Close help |

Reopening help starts a fresh search.

## File tree

| Key | Action |
|-----|--------|
| `Space` | Toggle expand directory |
| `Enter` | Expand directory / jump to file in diff |
| `o` | Expand all directories |
| `O` | Collapse all directories |
| `i` | Filter to files matching a regex (include) |
| `e` | Filter out files matching a regex (exclude) |
| `I` | Clear the include filter |
| `E` | Clear the exclude filter |
| `C-s` | Search file paths (substring); next match while a search is active |
| `C-r` | Previous file-path match |
| `C-g` | End the file-path search |

These keys are only active while the file tree is focused — in the diff, `i` still
edits the comment at the cursor and `C-s` still searches the diff.

### Filtering

`i` and `e` take case-insensitive regexes matched against each file's **full
relative path** (`^src/`, `\.rs$`, `test|spec`). Both can be active at once:
include runs first, then exclude removes from what's left.

A filter is not just a tree view — hidden files also disappear from the diff
pane, from `{`/`}` file navigation, `[`/`]` hunk navigation, and from the file
and `+/-` counts in the header. The tree title reports how much is hidden
(`Files · 2/12 · 12 of 58`), and the active patterns show in its bottom border.

Enter applies, `C-g` (or `Esc`) cancels, `Ctrl-u` clears the line. Reopening a prompt
pre-fills the pattern already applied, so submitting an emptied buffer is the
same as `I` / `E`. An invalid regex reports the error and leaves the prompt open
so it can be fixed. Comments on hidden files are not deleted and still export —
filters are a view, not an edit.

Filters are session-local: they are not written to the review session and reset
when tuicr restarts, but they survive a `:e` reload.

### Hiding reviewed files

`:set noreviewed` / `:set reviewed` / `:set reviewed!` (or the bare `:reviewed`)
hide or show files already marked reviewed with `r`. There is deliberately no
single-key binding: this is a command-only feature. Like
`i` / `e`, hidden files leave the tree, the diff pane, `{`/`}` and `[`/`]`
navigation, search, and the `+/-` counts in the header — so the header reports
the diff still left to review.

Two things stay deliberately unaffected. The tree title keeps counting reviewed
files in its `reviewed/total` fraction, since scoping it to the visible rows would
collapse it to `0/n` exactly when progress matters most; the bottom border carries
a `reviewed hidden` cue instead. And a file whose hunks are individually marked
with `R` is not hidden — only the file-level `r` flag counts.

While hiding, `r` becomes a burn-down loop: marking the file you are reading moves
you to the next unreviewed file, wrapping at the end. A hidden file cannot be
un-reviewed, because `r` can no longer reach it — `:set reviewed` brings it back.
Start a session with them hidden via `show_reviewed = false` in `config.toml`.

### Search

`C-s` moves only the tree selection to the next matching path, expanding collapsed
parents as needed, and leaves the diff viewport where it was — press `Enter` to
jump the diff there. `C-s` / `C-r` step matches and wrap around. `C-g` (or
submitting an empty prompt) ends the search, so the next `C-s` opens a new prompt. Search only ever
considers files that pass the active filters.

## Panel focus

| Key | Action |
|-----|--------|
| `Tab` / `Shift-Tab` | Cycle focus forward / backward between file list, comment navigator, diff, and commit selector |
| `<leader>h` | Move focus one panel left (side-by-side: new side → old side → file list) |
| `<leader>l` | Move focus one panel right (side-by-side: file list → diff → old side → new side) |
| `<leader>k` | Move focus up (comments to files, or diff/files to commit selector when visible) |
| `<leader>j` | Move focus down (files to comments when visible, otherwise diff) |
| `<leader>e` | Toggle file list visibility |
| `<leader>s` | Toggle commit selector visibility (also `:set commits!`) |
| `<leader>n` | Toggle relative line numbers (also `:set relativenumber!`) |
| `Enter` | Select file (when file list is focused) |

## Comment navigator

Shown below the file tree when local comments or visible remote PR threads exist.

| Key | Action |
|-----|--------|
| `C-n` / `C-p` | Move selection |
| `C-b` / `C-f` | Scroll rows left / right |
| `Enter` | Jump to selected comment |

## Review actions

| Key | Action |
|-----|--------|
| `r` | Toggle file reviewed |
| `R` | Toggle hunk reviewed |
| `c` | Add line comment (or file comment if not on a diff line) |
| `C` | Add file comment |
| `<leader>c` | Add review comment |
| `C-SPC` | Enter visual mode for range comments |
| `dd` | Delete comment at cursor |
| `i` | Edit comment at cursor (vim: text cursor at start) |
| `A` | Edit comment at cursor with text cursor at end (vim mode only) |
| `e` | Open focused file in `$EDITOR` |
| `y` | Copy review to clipboard |
| `Y` | Copy the comment at cursor to clipboard |

`e` opens the file at the cursor's line. Terminal editors (`vim`, `nvim`, `nano`, …)
take over the screen and tuicr reloads the diff once they exit. Windowed editors
(`code`, `cursor`, `zed`, `subl`, …) open in their own window while tuicr stays on
screen; reload with `:e` after editing. Adding `--wait` to the editor command opts a
windowed editor back into the blocking behaviour. Set the `editor` config key to
override `$EDITOR`.

In PR review `e` opens the revision under review, not whatever the checkout
happens to hold. When the local file *is* that revision you get the real file and
your edits land in it; otherwise — wrong branch checked out, file added or deleted
by the PR, no checkout at all — tuicr writes a read-only copy of the PR's content
to a temp directory and opens that, so the text and the line the cursor sits on
match the diff. The status bar names which one you got, e.g.
`Opened src/main.rs @ 1a2b3c4 (read-only PR copy)`. Binary files are only opened
from the checkout, never copied.

## Visual mode

| Key | Action |
|-----|--------|
| `C-n` / `C-p` | Extend selection down / up |
| `c` / `Enter` | Create comment for selected range |
| `s` | Start an editable suggestion prefilled with the selected code lines |
| `C-g` / `Esc` / `C-SPC` | Cancel selection |

Suggestions replace whole lines on the new side of a single diff hunk. Selections
across gaps (including deleted-only rows) or expanded context are rejected.
Unchanged context between added code blocks is included in the suggestion.
GitLab suggestions can include at most 100 lines above the final selected line.

## Comment mode

| Key | Action |
|-----|--------|
| `Tab` / `Shift-Tab` | Cycle comment type forward / backward (per `comment_types` order) |
| `Enter` / `Ctrl-Enter` / `Ctrl-s` | Save comment |
| `Shift-Enter` / `Ctrl-j` | Insert newline |
| `C-b` / `C-f` / `←` / `→` | Move cursor left / right |
| `C-p` / `C-n` / `↑` / `↓` | Move cursor up / down one displayed row (default editor) |
| `Ctrl-a` / `Ctrl-e` | Move to line start / end |
| `C-k` | Delete to the end of the line (at the end, join the next line) |
| `Ctrl-w` / `Alt-Backspace` / `Cmd-Backspace` | Delete word |
| `Ctrl-u` | Clear line |
| `C-g` / `Esc` / `Ctrl-c` | Cancel |

In the default editor, `C-p` / `C-n` move through wrapped rows and explicit newlines.
The cursor keeps its column where possible and restores it after crossing a
shorter row. Horizontal movement or editing sets the column for subsequent
vertical movement. `C-p` / `C-n` stop at the first or last row.

With `comment_vim = true` the box uses [`edtui`](https://github.com/preiter93/edtui)
modal editing (Normal/Insert/Visual: `hjkl`, `w`/`b`/`e`, `dd`/`D`/`ciw`/`x`,
`u`/`Ctrl-r`, visual `v`+`y`/`d`/`p`). From Normal mode `:w` (or `Enter` twice)
saves and `:q` (or `Esc`/`q` twice) cancels — the first press arms the action
and the header shows a confirm hint. `Alt-Enter` (Option+Enter) accepts and
`Alt-Esc` discards directly (no double-press) — Alt is the one modified
`Enter`/`Esc` that reaches the app across terminals, including browser/web
terminals like zellij web. `Tab` cycles the comment type in Normal
mode and inserts `comment_tab_width` spaces (default 4) in Insert mode; `Ctrl-s`
also saves.

## Commands

`M-x` opens the `:` command prompt; type a command below without the `:`.
In command mode, `Tab` and `Shift-Tab` complete or cycle command names.

| Command | Action |
|---------|--------|
| `:{N}` | Jump to new-side line N in current file |
| `:o{N}` | Jump to old-side line N in current file (matches deletions) |
| `:w` (`:write`) | Save session |
| `:e` (`:reload`) | Reload diff files |
| `:edit` | Open focused file in `$EDITOR` |
| `:clip` (`:export`) | Copy review to clipboard |
| `:copy-url` | Copy the open PR URL to clipboard (PR mode) |
| `:summary` | Show all pending local-draft comments; `C-n`/`C-p` select, `dd` deletes, and `Enter` jumps |
| `:diff` | Toggle diff view (unified / side-by-side) |
| `:theme` | Open the runtime theme picker (live preview; `C-s` filters, `Enter` applies and saves, `C-g` reverts) |
| `:theme <name>` | Apply and save `<name>` directly, without opening the picker |
| `:focus` (`:f`) | Toggle single-file view |
| `:stage` | Stage reviewed files (unstaged diffs only) |
| `:vim` (`:set vim!`) / `:novim` (`:set novim`) / `:set vim` | Toggle / disable / enable vim modal editing in the comment box (overrides `comment_vim`) |
| `:commits` (`:targets`) | Select commits to review |
| `:prs` | Open the review target selector on Pull Requests |
| `:sessions` (`:reviews`) | Resume a saved review for this checkout |
| `:submit` | Open submit picker (Comment / Approve / Request changes / Draft) |
| `:submit comment` | Submit a Comment review |
| `:submit approve` | Submit an Approve review |
| `:submit request-changes` | Submit a Request-changes review |
| `:submit draft` | Submit a Draft (pending) review |
| `:comments unresolved` | Show unresolved remote comments (PR mode, default) |
| `:comments all` | Show all remote comments including resolved and outdated |
| `:comments hide` | Hide remote comments in PR mode |
| `:set wrap` | Enable line wrap in diff view |
| `:set wrap!` (`:wrap`) | Toggle line wrap in diff view |
| `:set relativenumber` / `:set norelativenumber` | Enable / disable relative rendered-row numbers |
| `:set relativenumber!` | Toggle relative rendered-row numbers |
| `:set commits` | Show inline commit selector |
| `:set nocommits` | Hide inline commit selector |
| `:set commits!` | Toggle inline commit selector |
| `:set reviewed` | Show files already marked reviewed |
| `:set noreviewed` | Hide files already marked reviewed |
| `:set reviewed!` / `:reviewed` | Toggle files already marked reviewed |
| `:clear` | Clear all comments |
| `:clearc` | Clear comments without clearing reviewed marks |
| `:help` (`:h`) | Open the help screen |
| `:messages` | Open full details for the current error |
| `:version` | Show tuicr version |
| `:update` | Check for updates |
| `:q` (`:quit`) | Quit (warns on unsaved comments; discards review-only state) |
| `:q!` (`:quit!`) | Force quit |
| `:x` / `:wq` | Save and quit (prompts to copy if comments exist) |
| `ZZ` | Save and quit |
| `ZQ` | Quit without saving |
| `?` | Toggle help |

Pressing bare `q` no longer quits by default; it prints a reminder to use `M-x q` instead. Set
`q_quits = true` to restore `q` as a quit key in review modes.

The summary replaces the diff while leaving the file sidebar visible when it is open. The first
pending comment is selected when the summary opens. Use `C-n`/`C-p` to select the next
or previous comment; the view scrolls automatically to keep the selection visible. `Enter` returns
to the continuous diff and jumps to the selected comment, leaving single-file view if necessary,
while `C-g` returns without jumping. Reviewed files and hunks are revealed for the jump without
losing their reviewed state.
Press `dd` to delete the selected local draft, including comments whose original file or line
is absent from the current diff. Other comments are kept. Return with `C-g` and use `M-x w` to
save the deletion, following the existing diff-view deletion workflow.

The theme picker lists every bundled theme plus any local `*.toml` themes in the theme directory.
`C-n`/`C-p` move the selection and immediately repaint the UI with that theme (live preview) without
saving anything. `C-s` opens a filter prompt — type to narrow the list by substring, `Enter` applies
the filter and returns to navigation, `C-g` discards the filter edit without changing the list.
`Enter` on the picker itself keeps the currently previewed theme for the rest of the session. Like
other `:` toggles, the choice is not written to `config.toml`; set `theme = "<name>"` there to keep
it. `C-g` on the picker reverts the preview to whatever theme was active before it opened.

Not every forge supports every event:

| Event | GitHub | GitLab | Gitea | Bitbucket | Azure DevOps | Gerrit |
|---|---|---|---|---|---|---|
| `comment` | yes | yes | yes | yes | yes | yes |
| `approve` | yes | yes | yes | yes | yes (vote +10) | yes (vote +2) |
| `request-changes` | yes | yes | yes | no | yes (vote -10) | yes (vote -1) |
| `draft` | yes | yes | yes | no | yes (plain comment) | yes |

Bitbucket rejects `request-changes` and `draft` up front rather than silently downgrading them.
Azure DevOps has no pending-review primitive, so `draft` posts as a plain comment with no vote.
Gitea requires a review summary for `request-changes` and `draft`, and for `comment` when there
are no inline comments; inline comments alone do not satisfy it. GitLab `draft` creates draft
notes that the author publishes from GitLab's own "Submit review" UI. Gerrit `draft` stores draft
comments that the author publishes from Gerrit's Reply UI.

## Commit selection / review target selector

| Key | Action |
|-----|--------|
| `Tab` / `Shift-Tab` | Switch between Local, Pull Requests, and Sessions tabs |
| `C-n` / `C-p` | Move selection |
| `Space` | Toggle local commit selection |
| `Enter` | Confirm local commit range, open PR, load more PRs, or resume a saved review |
| `C-s` | Filter currently loaded PR rows locally |
| `r` | In Pull Requests tab, toggle all open PRs / PRs requesting your review |
| `a` | In Sessions tab, show or hide saved reviews holding no progress |
| `dd` | In Sessions tab, delete the saved review under the cursor (asks to confirm) |
| `C-g` / `Esc` | Return to the diff |
| `M-x q` | Quit |

The Sessions tab lists saved reviews for the current checkout, so you can pick one
instead of retyping the commit range it was opened with. Rows show the review target,
comment count, reviewed files, and age. Reviews with no comments and no reviewed files
are omitted; press `a` to show them. Selecting a saved PR review re-fetches it from the
forge, the same as opening it from the Pull Requests tab.

Press `dd` to delete the saved review under the cursor. tuicr asks to confirm first, and
refuses to delete a review that is open in another tuicr, since that process would write
it straight back. Combine `a` with `dd` to clear out empty sessions left behind by a
crash, or by `tuicr review delete`/`clear` emptying a review another tuicr had open.

## Inline commit selector

Shown at the top of the diff when reviewing multiple commits. Focus it with `<leader>k` or `Tab`.
When opening a GitHub PR or GitLab MR you have reviewed before, tuicr may preselect only commits
newer than your latest submitted review; commits already covered by that review are marked with
`✓`. Use `Space` / `Enter` here to expand or adjust the range. Bitbucket does not record which
commit an approval covered, so no commits are preselected there.

| Key | Action |
|-----|--------|
| `C-n` / `C-p` | Navigate commits |
| `Space` / `Enter` | Toggle commit selection (updates diff) |
| `(` / `)` | Cycle through individual commits |
| `C-g` / `Esc` | Return focus to diff |

## Confirm dialogs

| Key | Action |
|-----|--------|
| `y` / `Enter` | Yes |
| `n` / `C-g` / `Esc` | No |

## Mouse

Mouse support is on by default. Disable with `mouse = false` in config.

| Action | Effect |
|--------|--------|
| Wheel up / down | Scroll the panel under the cursor (file list, comment navigator, diff, commit list, or help popup) without moving the cursor line |
| Click on a file | Jump to that file (lazygit-style) |
| Click on a directory | Expand or collapse it |
| Click on a diff line | Position the cursor on that line |
| Click on a commit | Toggle selection (or expand the row to load more) |
| Drag in diff | Highlight a range; press `y` to copy the selected source lines |

For full native terminal selection across the UI, hold your terminal's bypass modifier while dragging (usually **Shift** or **Option/Alt**, depending on the terminal).
