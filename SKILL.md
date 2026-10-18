---
name: measure-or-abstain
description: Prevent confident wrong conclusions built on empty tool output. Use when a command exits 0 with no output, when a probe says "device not found / no files / nothing detected", when a sandbox may be hiding things from a tool, or when a check returns zero and you are about to report "absent" or "clean". Teaches positive controls and a three-state result (present / absent / could-not-measure).
---

# measure-or-abstain

Empty output is **not** "absent". It is "absent *or unmeasurable*". Treat them differently.

## The incident

In a sandbox, `system_profiler SPUSBDataType` exits 0 with zero lines. The whole USB tree is empty -
keyboard and mouse too, not just the phone. The agent reported "phone not plugged in", then stacked an
interpretation on the empty value ("so it must be wireless") and advised "no need to plug it in".
There was no measurement, yet there was a conclusion - and then advice built on the conclusion.

## Rules

1. **Positive control.** When you test for something's absence, also measure something that MUST be
   present (a keyboard, another data type, a file you just wrote). If the control is missing too,
   the tool is blind. Report `could-not-measure`.
2. **Three states, not two.** Return `present`, `absent` (control present, target missing), or
   `unmeasurable`. Never map "no output" to `absent`.
3. **A check that measured zero items must not be green.** "No violations over 0 files" is not clean.
   (Same failure as a vacuous lint gate.)
4. **Prefer the tool that answers the actual question** over one that merely correlates:
   - iOS device connected? Ask `xcrun devicectl device info details --device <udid>` and read
     `transportType` (wired/wireless), not USB enumeration.
5. **Do not stack inference on an unmeasured value.** If step 1 is "unknown", step 2 starts with
   "unknown", not with a guess that sounds consistent.
6. **Say what you could not measure** in the report, in one line.

## Snippet (shell)

```bash
# probe <control-command> -- <target-command>   prints present | absent | unmeasurable
# "absent" is only ever printed when the CONTROL answered and the TARGET answered with nothing.
probe() {
  local ctl=() tgt=() seen=0 a
  for a in "$@"; do if [ "$a" = "--" ]; then seen=1; elif [ $seen = 0 ]; then ctl+=("$a"); else tgt+=("$a"); fi; done
  [ ${#ctl[@]} -gt 0 ] && [ ${#tgt[@]} -gt 0 ] || { echo "usage: probe <control...> -- <target...>" >&2; echo unmeasurable; return; }
  local cout; cout="$("${ctl[@]}" 2>/dev/null)" || { echo unmeasurable; return; }      # the control must succeed...
  [ -n "$cout" ] || { echo unmeasurable; return; }                                       # ...and must print something: a blind tool prints nothing for it too
  local tout terr rc; tout="$("${tgt[@]}" 2>/tmp/probe.$$.err)"; rc=$?; terr="$(cat /tmp/probe.$$.err)"; rm -f /tmp/probe.$$.err
  if [ -n "$tout" ]; then echo present
  elif [ $rc -eq 0 ] || { [ $rc -eq 1 ] && [ -z "$terr" ]; }; then echo absent         # empty output, no error text (grep-style "no match" is exit 1)
  else echo "unmeasurable"; [ -n "$terr" ] && echo "  reason: $terr" >&2; fi
}
# examples:
probe ls / -- ls /some/dir            # absent only if / listed fine and /some/dir exists but is empty
probe system_profiler SPHardwareDataType -- system_profiler SPUSBDataType   # a blind sandbox fails the control or the target and reports unmeasurable
```
