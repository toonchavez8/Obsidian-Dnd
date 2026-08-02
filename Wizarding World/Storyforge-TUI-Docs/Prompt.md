I need you to help me update four implementation guides, not write code into the code repo.

I will provide:
1. The end-goal game sheet - this will only be for refernce.
2. Access to read the current game/code status from another repo.
3. The four existing guides that need to be updated - you will only update the guide to cover only what the guide intially covers even if the end-goal game sheet expects more this is a work in progess you are hellping me make that progress.

Your job is to read all of that context, then update the four guides so they form a literal step-by-step learning path from the current state in a progressive walk to the end goal, i do not expect we will reach it in the four guides, .

Important constraints:
- Do not edit the game/code repo.
- Only write into the guide files.
- Preserve existing notes inside code blocks unless they are clearly obsolete.
- Expand the guides with what has already been done, why it was done, and what should happen next.
- The guides must be written so I can manually implement the code myself and learn from the process.
- Do not give vague instructions like “create four helper functions.”
- Instead, explain the logic incrementally and concretely.

For every step, use this structure:

File: `path/to/file.rs`

Step X: short action title

Current state:
Explain what already exists and what it currently does.

Why this step matters:
Explain the purpose of the change in plain language.

Before:
Show the relevant existing code block or describe exactly where the code belongs.

What to change:
Give the specific manual change I should make.

Learning checkpoint:
Explain what I should understand before moving on.

Temporary MVP / debug behavior:
If the full feature is too large, guide me through a small testable version first.
For example:
- If props/arguments are passed into a function, first log them to the terminal or TUI.
- If a function adds two numbers, first log both numbers.
- Then perform the addition and log the sum.
- Then return the sum.
- Then connect it to the next part of the flow.

After:
Show the expected final code block for that step.

How to verify:
Explain exactly what I should run, click, or observe to confirm the step worked.

Next connection:
Explain how this step connects to the next function, module, screen, or guide.

The guide should teach the logic in an MVP / iterative flow:
1. Confirm data reaches the function.
2. Log the input.
3. Perform the smallest useful operation.
4. Log the result.
5. Return or display the result.
6. Connect it to the next part of the app.
7. Only then expand the behavior.

When multiple functions are involved, name them explicitly.
For example:
“The flow uses four functions: `a`, `b`, `c`, and `d`.
First we prove `a` receives data.
Then we prove `a` can pass data to `b`.
Then `b` transforms the data.
Then `c` prepares it for display.
Then `d` renders or stores the final result.”

Use the `$human-writing` and `$rust-patterns` skills while preparing the guide:
- `$human-writing`: make the guide clear, plainspoken, and easy to follow as a learning document.
- `$rust-patterns`: use idiomatic Rust explanations, ownership/borrowing notes where relevant, and Rust-appropriate examples.

If either skill is unavailable in the current environment, say so briefly, then continue using the closest fallback approach.

Output requirements:
- Update all four guides in the correct logical order.
- The guides should continue to cover the overall logic of progression towards the end goal
- Make the order explicit at the top of the first guide.
- At the beginning of every step, name the exact file path that needs to be changed.
- Include before/where/after code blocks when useful.
- Keep the guide practical and implementation-focused.
- Do not implement the code yourself.
- Do not skip reasoning steps.
- Do not assume I already know why the code is being changed.

1. Overall End-goal Game sheet: C:\Users\toonc\Documents\Obsidian-Dnd\Wizarding World\Storyforge-TUI-Docs\DND-rpg-ww-tui-game-sheet.md 

2. The repo: C:\Users\toonc\Dev\82-Storyforge-TUI

That is the current state of our code, that is where i will write the code, not you

The game guides:
- C:\Users\toonc\Documents\Obsidian-Dnd\Wizarding World\Storyforge-TUI-Docs\Guide\04-responsive-dashboard.md
- C:\Users\toonc\Documents\Obsidian-Dnd\Wizarding World\Storyforge-TUI-Docs\Guide\05-engine-command-event-model.md
- C:\Users\toonc\Documents\Obsidian-Dnd\Wizarding World\Storyforge-TUI-Docs\Guide\06-content-pack-format.md
- C:\Users\toonc\Documents\Obsidian-Dnd\Wizarding World\Storyforge-TUI-Docs\Guide\07-character-creation.md