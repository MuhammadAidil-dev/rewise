# Installing Rewise

These steps are written for an AI agent. Run them from the **root of the
project** that should get a memory. Do not modify the Rewise skill itself.
Paths like `PROTOCOL.md` and `templates/` below are relative to the skill's
own directory (the folder that contains this file).

Rules for the whole install:

- Never overwrite an existing file. If something already exists, skip it.
- Do not delete anything.
- Report what you created and what you skipped when you finish.

## Steps

1. **Create the memory folders** (skip any that exist):
   - `memory/`
   - `memory/mistakes/`
   - `memory/lessons/`
   - `memory/decisions/`

   Git does not track empty folders. Add an empty `.gitkeep` to each of the
   three subfolders so they survive a commit.

2. **Create `memory/index.md`** from `templates/index.md`, only if it does not
   exist.

3. **Copy `PROTOCOL.md`** from the kit to `memory/PROTOCOL.md`, only if it does
   not exist. The rules then travel with the project and need no external
   lookup.

4. **Add the always-on block** from `templates/AGENTS-block.md`:
   - If `AGENTS.md` exists, append the block at the end.
   - Else if `CLAUDE.md` exists, append the block there.
   - Else create `AGENTS.md` containing only the block, and tell the user.
   - If the file already contains a heading `## Rewise`, do not add it again.

5. **Report**: list created files, skipped files, and which file received the
   block.

## Verify

- `memory/index.md`, `memory/PROTOCOL.md`, and the three subfolders exist.
- Exactly one `## Rewise` block exists in the project's rules file.
- No pre-existing file lost content.

## Uninstall

Delete the `memory/` folder and the `## Rewise` block from `AGENTS.md` /
`CLAUDE.md`.
