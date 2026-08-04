---
name: cowork-windows-fix-guide
description: >
  Generate a client-facing, step-by-step guide for fixing Claude Cowork when it won't start,
  hangs, or errors on a Windows computer. The common documented cause is the Windows "Virtual
  Machine Platform" feature (and its vmcompute/hns services) being turned off, which Cowork
  depends on. Use when someone you support can't get Cowork working on Windows, or you're asked
  for "the Cowork Windows fix," "the Windows guide to send a client," or similar. Produces the
  guide as chat text, and as a Word document to hand over when a docx skill is available.
---


# Cowork Windows Fix — Client Guide Generator

## When to use this
Trigger whenever someone you support can't get Claude Cowork working on Windows, or you're asked for "the Cowork Windows guide," "the fix to send to [client]," or similar. This covers the confirmed root cause: Cowork on Windows requires the Windows "Virtual Machine Platform" feature (and its `vmcompute`/`hns` services) to be enabled, per Anthropic's own support docs (support.claude.com articles "Get started with Claude Cowork" and "Deploy Claude Desktop for Windows").

## What to produce
Produce the guide text below **verbatim** (light formatting/headers are fine; do not alter the steps, commands, or ordering).

- **Always** paste the full guide into the chat response, so it can be copy-pasted directly into an email.
- **If a docx skill is available** in the current environment (read its SKILL.md first), also generate a Word document (.docx) titled **"Fixing Claude Cowork on Windows — Step-by-Step.docx"** containing the same guide. Save it to a connected folder if one is available, otherwise to the outputs folder, and share it via whatever file-presentation mechanism the runtime exposes (e.g. `mcp__cowork__present_files` in Cowork). If no docx skill is available, the chat version alone is sufficient — don't block on it.

Do not add extra troubleshooting steps beyond what's below — this list is deliberately scoped to the confirmed, documented cause and kept simple for a non-technical reader. If the reported symptom doesn't match this cause (e.g., billing/plan eligibility, macOS, or a completely different error), flag that this specific guide may not apply rather than stretching it to fit.

## Guide content (use verbatim)

---

# Fixing "Claude Cowork Not Working" on Windows

This guide walks through the most common cause of Claude Cowork failing to start, hanging, or showing an error on a Windows computer: a Windows feature called **Virtual Machine Platform** needs to be turned on. Cowork depends on it to run.

Please work through the steps in order, and don't skip ahead — each step tells you what to check before moving to the next.

## Before you start

You'll use a tool that's already built into Windows called **PowerShell**. You don't need to download anything. A couple of steps ask you to run PowerShell "as Administrator" — that just means giving it permission to change system settings, the same kind of permission you'd give any program during setup.

## Step 1: Open PowerShell as Administrator

1. Click the **Start button** (the Windows logo) in the bottom-left corner of the screen.
2. Type the word **PowerShell**. You'll see "Windows PowerShell" appear in the search results.
3. **Right-click** on "Windows PowerShell" — don't left-click to open it normally.
4. Click **"Run as Administrator"** from the menu that appears.
5. A box will likely pop up asking "Do you want to allow this app to make changes to your device?" Click **Yes**.
6. A window will open with a blinking cursor — this is PowerShell. Keep it open; you'll use it for the next steps.

## Step 2: Check whether the required feature is switched on

1. Click inside the PowerShell window so your cursor is active there.
2. Type or paste this exactly, then press **Enter**:

   ```
   Get-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform
   ```

3. Wait a few seconds. Information will appear, including a line that reads **"State :"** followed by either **Enabled** or **Disabled**.
4. Check what it says:
   - If it says **State : Enabled** → skip ahead to **Step 4**.
   - If it says **State : Disabled** → continue to **Step 3**.

## Step 3: Turn the feature on (only if Step 2 said "Disabled")

1. In the same PowerShell window, type or paste this exactly, then press **Enter**:

   ```
   Enable-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform -All
   ```

2. This takes a minute or two. Let it finish — the cursor will return when it's done.
3. **Restart the computer** — this is important, and it must be a **Restart**, not a **Shut down**:
   - Click the **Start button** → click the **Power icon** → click **Restart**.
   - Do not shut the computer down and turn it back on manually. Windows has a setting called "Fast Startup" that can leave the fix incomplete if you shut down instead of restarting. A proper Restart avoids this.
4. Once the computer turns back on, repeat **Step 2** to confirm it now says **Enabled**.

## Step 4: Check that two background services are running

1. If PowerShell is closed, open it as Administrator again (repeat Step 1).
2. Type or paste this exactly, then press **Enter**:

   ```
   Get-Service vmcompute, hns
   ```

3. You'll see two rows of results. Check the **Status** column for both:
   - Both should say **Running**.
   - If either says **Stopped**, or you see a message saying it "cannot find" the service, stop here — this needs a different fix. Take a screenshot of what you see and send it back.

## Step 5: If it still doesn't work — check how the computer was restarted

If Steps 2 and 4 both looked correct but Cowork still won't open, ask yourself: have you been **shutting the computer down** and turning it back on, rather than **restarting** it?

Windows' "Fast Startup" feature can leave things half-loaded after a shutdown, even if everything is set up correctly. Do a full **Restart** (Start → Power → Restart) and try Cowork again.

## Step 6: Pull the diagnostic log (only if still stuck after all of the above)

1. Open the **Claude Desktop** app.
2. Click **Help** in the menu, then **Troubleshooting**, then **Show Logs**.
3. This opens a folder on your computer. Look for a file named **supported-features-info.json**.
4. Send that file (or a screenshot of its contents) back — it shows exactly which check is failing, so the next step can be pinpointed rather than guessed at.

## What to report back after each step

- **After Step 2:** what the "State" line said (Enabled / Disabled).
- **After Step 4:** what both "Status" values said (Running / Stopped / not found).
- **If you reach Step 6:** attach the `supported-features-info.json` file.

---

## Source

This guide reflects the officially documented fix from Anthropic's Claude Help Center: "Get started with Claude Cowork" (support.claude.com/en/articles/13345190) and "Deploy Claude Desktop for Windows" (support.claude.com/en/articles/12622703), which specifically covers the "Missing HCS services" / Virtual Machine Platform error.
