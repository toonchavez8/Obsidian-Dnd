# 23: Repeat the documentation review loop

## Result

You can review the guide one chapter at a time, find instructions that force a junior developer to invent missing behavior, repair them, and record what was verified. The loop is small enough to run after every engine milestone.

The current review state lives in `../REVIEW-LEDGER.md`.

## What “step by step” means

A chapter is not step by step merely because it uses a numbered list. A complete implementation unit answers all of these questions:

| Question | Required evidence |
| --- | --- |
| Where does this go? | Exact repository-relative file path |
| Why does it exist? | One short responsibility statement |
| What do I write? | Complete type, function, data record, or configuration |
| What enters it? | Parameters or command payload |
| What can fail? | Validation rule and player-safe error |
| What changes? | Emitted event and reducer behavior |
| Where is it connected? | Import, module export, dispatcher, or renderer wiring |
| How do I prove it? | Focused automated test |
| How do I see it? | Manual TUI action and expected behavior |
| When may I continue? | Checkpoint command and passing result |

A design-only section must say `Design only: implemented in chapter NN`. Otherwise the reader should be able to perform the step now.

## The review cycle

Review one file through all five passes before opening the next file.

```text
inventory
   |
   v
structure pass
   |
   v
code-contract pass
   |
   v
continuity pass
   |
   v
junior read-through
   |
   v
verification and ledger update
   |
   +-------- problems found --------+
   |                                |
   +---------- repair and rerun <---+
```

### Pass 1: Inventory the chapter

From the documentation root:

```powershell
Get-ChildItem -LiteralPath Guide -Filter '*.md' |
    Sort-Object Name |
    Select-Object -ExpandProperty Name
```

For the selected chapter, list its headings and code-fence count:

```powershell
$chapter = 'Guide\10-tactical-combat-mvp.md'
rg -n '^#{1,6} ' $chapter
$fences = (Select-String -LiteralPath $chapter -Pattern '^```').Count
"Code fences: $fences"
```

The fence count must be even. An odd value means a code block was not closed.

### Pass 2: Find naked implementation lists

Search for phrases that often hide missing work:

```powershell
rg -n -i '^(add|create|implement|define|support)( these)?:?$|add commands|add events|add methods|core functions' Guide --glob '!23-documentation-review-loop.md'
rg -n -i 'later|eventually|as needed|and so on|similar methods|remaining methods' Guide --glob '!23-documentation-review-loop.md'
```

The phrase itself is not always wrong. Inspect the following lines. A list of commands such as `Move`, `CastSpell`, and `EndTurn` is incomplete unless the chapter also provides:

- The Rust enum and each payload.
- Validation before mutation.
- Emitted event payloads.
- Reducer changes.
- Rejection behavior.
- Focused tests.

Use this repair order:

1. Add the exact destination file.
2. Add the complete enum or function.
3. Add a behavior table.
4. Add handler or helper code.
5. Add reducer code.
6. Add a focused test.
7. Add the narrow run command.
8. Add expected behavior.

Do not respond to a large naked list by adding one very large function. Break it into testable helpers whose names describe one rule.

### Pass 3: Check continuity with earlier chapters

Search the whole documentation set before introducing a new name:

```powershell
rg -n 'GameCommand|GameEvent|ContentId|SpellSlotState|JourneyState' .
```

For every reused type, confirm:

1. It was introduced in an earlier chapter.
2. Its field names still match.
3. New variants are described as extensions, not silent replacements.
4. Serialization changes have a save migration.
5. The TUI does not duplicate a core calculation.

When a later chapter changes an earlier rule, update both chapters in the same documentation commit. The spell-slot conversion is the example: the game sheet, character creation, combat, travel-rest rules, saves, and UI all had to stop referring to mana.

### Pass 4: Perform a junior read-through

Read the chapter from the position of someone who has never built a game.

At each code block, ask:

- Do I know which file is open?
- Is this a new file or an edit?
- Do I know what existing code remains?
- Are every imported type and crate available by this chapter?
- Is a deliberately incomplete example labeled?
- Does the next command compile only this change?
- Does the expected result tell me whether I succeeded?

At each rules section, ask:

- Can two reasonable developers implement different behavior from these words?
- If so, should the decision be campaign data, a rules-profile option, or one explicit engine rule?
- Does the guide explain that ownership?

At each manual test, ask:

- What seed and content pack should I use?
- What exact action triggers the feature?
- What visible result proves it worked?
- What failed action proves validation is safe?
- Does quit and reload preserve it?

Repair the first unclear point before continuing. Later confusion often comes from that first missing contract.

### Pass 5: Verify and update the ledger

Run the documentation scans:

```powershell
rg -n -i '\bmana\b' . --glob '!Guide/23-documentation-review-loop.md'
rg -n 'campaigns-private/wizarding-world-private' . --glob '!Guide/23-documentation-review-loop.md'
rg -n -i 'todo|fixme|write this later|implementation omitted' . --glob '!Guide/23-documentation-review-loop.md'
```

Expected result: no active rule uses mana, no launch command points at an in-repository private pack, and no chapter claims omitted implementation is complete.

Check local Markdown links with this PowerShell snippet:

```powershell
$docsRoot = (Resolve-Path '.').Path
$brokenLinks = [System.Collections.Generic.List[string]]::new()

Get-ChildItem -LiteralPath $docsRoot -Filter '*.md' -Recurse | ForEach-Object {
    $source = $_
    $text = Get-Content -LiteralPath $source.FullName -Raw
    [regex]::Matches($text, '\[[^\]]+\]\(([^)#]+)(?:#[^)]+)?\)') | ForEach-Object {
        $target = [System.Uri]::UnescapeDataString($_.Groups[1].Value)
        if ($target -notmatch '^(https?|mailto):') {
            $resolved = Join-Path $source.DirectoryName $target
            if (-not (Test-Path -LiteralPath $resolved)) {
                $brokenLinks.Add("$($source.FullName): $target")
            }
        }
    }
}

if ($brokenLinks.Count -gt 0) {
    $brokenLinks
    throw "$($brokenLinks.Count) broken local Markdown link(s)"
}

'Local Markdown links passed'
```

Then update the selected file's row in `REVIEW-LEDGER.md`:

- `Structure` after every result, checkpoint, verification, and acceptance section is present.
- `Contracts` after every implementation list has payloads and behavior.
- `Continuity` after names and rules match the rest of the guide.
- `Junior pass` after a beginning developer can follow it without inventing hidden code.
- `Runtime` only after the code repository reaches that chapter and the commands actually pass.

Do not mark `Runtime` complete from reading alone.

## Repair example: combat commands

Before repair:

```text
Add commands:
Move
Cast
Defend
EndTurn
```

This leaves the encounter ID, actor, target, validation, events, and failure state undefined.

After repair, chapter 10 provides:

- `CombatCommand` with an `EncounterId`, `ActorId`, and command-specific payload.
- `CombatEvent` with structured rolls and resource changes.
- A behavior table for all commands.
- A legality checklist that runs before mutation.
- Tests proving a rejected cast spends no action, slot, or sorcery point.

That is the standard to apply to every similar list.

## Reviewing data-driven content

External RON or TOML is executable game design. Review it with the same care as Rust:

1. Provide a complete original record.
2. Explain every non-obvious field.
3. Name each cross-reference that must resolve.
4. Add validation diagnostics with stable codes.
5. Add one valid fixture and one invalid fixture.
6. State which content is public, private, or reference-only.
7. Never copy prose from a private PDF into the public demo.

For travel events, check the card has an objective, pillars, opening hook, scene, conditions, weight, cooldown, story beats, clues, rewards, and difficulty variants.

## When the implementation repository exists

Documentation code cannot be fully proven until it is placed in the Rust repository. At each chapter checkpoint:

```powershell
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features --locked -- -D warnings
cargo test --workspace --all-features --locked
cargo test --workspace --doc --locked
```

For content chapters:

```powershell
cargo run -p storyforge-tui -- validate --pack campaigns/academy-demo
```

For UI chapters, complete the manual terminal-size and exit-restoration tests. A snapshot alone does not prove the terminal was restored.

If guide code fails:

1. Keep the failing chapter branch.
2. Record the exact compiler or test error in the ledger note.
3. Fix the guide and implementation together.
4. Rerun the narrow test.
5. Rerun the workspace checks.
6. Mark `Runtime` only after both repositories contain the correction.

## Review cadence

Run the complete loop:

- After every guide chapter is implemented.
- Before each milestone tag.
- After changing a cross-cutting rule such as spell resources, time, saves, or IDs.
- Before the first public npm release.
- After a reader reports a missing step.

At milestone review, start at chapter 00 and stop at that milestone's final chapter. Later chapters do not need to block an earlier playable slice.

## Acceptance check

- Every documentation file has a ledger row.
- A command or event list cannot pass review without payload and behavior contracts.
- Cross-chapter terminology scans pass.
- Local Markdown links resolve.
- Design-only work identifies its implementation chapter.
- Runtime verification remains visibly separate from editorial review.
- A new reader can report the exact checkpoint where they became blocked.

## Suggested commit

```powershell
git add Storyforge-TUI-Docs
git commit -m "Add the repeatable guide review loop"
```
