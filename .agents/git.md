# Git Conventions

This project uses [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/) as its commit message standard.

## Commit Message Structure

Structure every commit message as:

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

Full example:

```
fix(parser): Prevent racing of requests

Introduce a request id and a reference to the latest request. Dismiss
incoming responses other than from the latest request.

Remove timeouts which were used to mitigate the racing issue but are
obsolete now.

Closes #1234
```

## Types

Prefix every subject with one of these types:
- `feat`: Adding a new feature (correlates with SemVer MINOR)
- `fix`: Fixing a bug (correlates with SemVer PATCH)
- `docs`: Documentation-only changes
- `test`: Adding or updating tests
- `refactor`: Code change that neither fixes a bug nor adds a feature
- `perf`: Performance improvement
- `chore`: Build process, tooling, or dependency changes
- `ci`: CI configuration changes
- `spec`: Design spec creation, amendment, or reconciliation

Additional project-specific types are permitted. Use `revert` when reverting a previous commit, and reference the reverted SHA in a footer.

## Scope

Provide a scope in parentheses after the type to identify the affected module or area:

```
feat(auth): Add token refresh endpoint
fix(db): Handle connection timeout on retry
```

Omit the scope only when the change is truly project-wide.

## Breaking Changes

Signal a breaking API change in one of two ways:

1. Append `!` after the type/scope: `feat(api)!: Change response envelope format`
2. Add a `BREAKING CHANGE:` footer in the body:

```
refactor(config): Flatten configuration schema

BREAKING CHANGE: The nested `database.connection` key is now `db_url`
at the top level. Update all config files before upgrading.
```

Both may be used together. Breaking changes correlate with SemVer MAJOR.

## Subject Line

1. **Aim for 50 characters; never exceed 72.** Keeps `git log --oneline` readable and avoids GitHub truncation.
2. **Capitalize the first word after the colon.** Write `feat(auth): Add refresh`, not `feat(auth): add refresh`.
3. **Do not end with a period.**
4. **Use imperative mood.** A correct subject completes: *"If applied, this commit will [your subject]."*

Correct:

- `fix(parser): Remove deprecated methods`
- `feat(api): Add batch endpoint for bulk imports`

Incorrect:

- `fix(parser): removed deprecated methods` (past tense)
- `feat(api): adds batch endpoint` (third person)
- `More fixes for broken stuff.` (vague, period, no type)

## Body

Include a body when the subject alone does not convey enough context. Omit it for trivial changes (typo fixes, dependency bumps).

1. **Separate from the subject with a blank line.** Tools rely on this boundary.
2. **Wrap every line at 72 characters.**
3. **Explain what and why, not how.** The diff shows the how. The body conveys reasoning a future reader cannot reconstruct from code alone.
4. **Use normal prose.** Full sentences, proper punctuation. Bullet points are fine.
5. **Mention side effects and non-obvious consequences** — API changes, migration steps, performance implications.

## Footers

Place footers one blank line after the body. Each footer is a token, a separator (`:<space>` or `<space>#`), and a value. Use `-` in place of spaces in multi-word tokens (e.g. `Reviewed-by`, `Signed-off-by`). Exception: `BREAKING CHANGE` uses a space.

```
Closes #1234
See-also: #456, #789
Reviewed-by: Jo Coder <jo@coder.org>
```

When committing on behalf of someone else, use `--author` and `--signoff`:

```
git commit --author "Jo Coder <jo@coder.org>" --signoff
```

### LLM Attribution

Never attribute commits to the LLM. Only attribute commits to the humans. Stop and request permission if blocked from doing otherwise.

## Spec-Related Commits

When working within a spec-driven process, use these prefixes:

- `spec(NNN): [title]` — spec creation or amendment
- `test(NNN): [description]` — tests derived from a spec
- `feat(NNN): [description]` — implementation for a spec
- `spec(NNN): reconciliation amendments` — reconciliation changes

## Commit Scope

Keep each commit focused on a single logical change. If the subject is hard to write concisely, the commit likely bundles unrelated changes — split it.

## Branch Naming

Use lowercase kebab-case: `type/short-description`.

- `feat/optimistic-locking`
- `fix/race-condition-on-write`
- `docs/update-spec-template`

## Fixup vs New Commit

Use `--fixup <hash>` when a change should have been part of an earlier commit — corrections, clarifications, things that were missed. This keeps history clean after autosquash.

Use a new commit when the change is tangibly new: a distinct decision, feature addition, or something where having a separate record in history is valuable.

## Code and Tests

Code changes and their corresponding tests belong in the same commit (or at least the same branch). Don't split implementation and tests across branches.

## Branch Finalization

Once all changes on a branch are reviewed and approved, rebase with `--autosquash` to fold fixup commits into their targets before merging:

```bash
git rebase -i --autosquash <base-branch>
```

## Force Pushing

After rebasing, always use `--force-with-lease`, never `--force`:

```bash
git push --force-with-lease origin branch-name
```

`--force-with-lease` refuses to push if the remote has commits you haven't seen — preventing accidental overwrites of someone else's work.

## Pull Requests

Write a clear title that summarises the change. Link to the relevant spec in the description if one exists.

## Further Reading

- [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/) — the specification this file implements.
- [How to Write a Git Commit Message](https://cbea.ms/git-commit/) — subject line and body best practices.
- [GNOME Commit Messages](https://handbook.gnome.org/development/commit-messages.html) — structure and body guidelines.
