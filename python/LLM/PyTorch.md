# 1 Weights only load failed.

Got it — this error message is from **PyTorch ≥2.6** when you try to `torch.load()` a checkpoint (like Bark’s model weights).

### Why you’re seeing it

- In PyTorch 2.6, `torch.load()` became **more restrictive by default**.
    
- Before 2.6, `torch.load(file)` used `weights_only=False` (full pickled objects, including arbitrary Python code).
    
- In 2.6+, `torch.load()` defaults to `weights_only=True` (only Tensors, safe loading).
    
- Bark’s checkpoint apparently contains **pickled Python objects** (e.g., `numpy.core.multiarray.scalar`) that are _not_ on PyTorch’s new safe allow-list.
    
- Result: the default safe unpickler refuses to load it and throws a `WeightsUnpickler error`.
    

### What the error means

- The checkpoint contains something beyond just tensor weights.
    
- Loading it with `weights_only=True` fails because PyTorch blocks unpickling arbitrary objects.
    
- Loading with `weights_only=False` would work but can run arbitrary code from the checkpoint (dangerous if you don’t trust it).
    

### How to fix

You have **two options** (only do this if you trust the file’s source):

---

#### **Option 1: Use `weights_only=False`**

```python
import torch

ckpt = torch.load("bark_checkpoint.pt", weights_only=False)  # unsafe but loads everything
```

This restores the old behaviour.  
⚠️ Only safe if the checkpoint is from a **trusted source**.

---

#### **Option 2: Keep `weights_only=True` but allow specific globals**

If you want to stay in “safe” mode, explicitly allow the blocked classes:

```python
import torch
import numpy as np

torch.serialization.add_safe_globals([np.core.multiarray.scalar])

ckpt = torch.load("bark_checkpoint.pt", weights_only=True)  # still safe but now allowed
```

or use the context manager:

```python
import torch
import numpy as np

with torch.serialization.safe_globals([np.core.multiarray.scalar]):
    ckpt = torch.load("bark_checkpoint.pt", weights_only=True)
```

This whitelists only `numpy.core.multiarray.scalar` so unpickling proceeds without opening up arbitrary code execution.

---

### In short

- PyTorch 2.6 made checkpoint loading more secure.
    
- Your Bark checkpoint triggers the new safe loader.
    
- Either revert to `weights_only=False` (old behaviour, riskier) **or** explicitly allow the required classes with `add_safe_globals`/`safe_globals`.
    

## in Bark

Here’s how you can configure / patch Bark’s code so it works under PyTorch 2.6+ and avoids the “Weights only load failed” error. Do this only if you trust the checkpoint file.

---

## What you need to change

In the Bark loader (or wherever the checkpoint is loaded), you need to modify the `torch.load(...)` invocation so that:

- Either you disable the “weights only” safe restriction (`weights_only=False`), or
    
- You explicitly allow the blocked global(s) (e.g. `numpy.core.multiarray.scalar`) via PyTorch’s safe globals API.
    

---

## Steps to patch Bark

Here is a sketch of what to do in the Bark repository/codebase.

1. **Find** the place where the checkpoint is loaded.  
    Likely something like:
    
    ```python
    checkpoint = torch.load(ckpt_path, map_location=device)
    ```
    
    Or possibly `torch.load(ckpt_path, map_location=device, weights_only=True)`.
    
2. **Modify** it to either:
    
    ### Option A — Set `weights_only=False`
    
    ```python
    checkpoint = torch.load(ckpt_path, map_location=device, weights_only=False)
    ```
    
    This tells torch to allow loading pickled objects, not just raw tensors. But it's less safe if the checkpoint is untrusted.  
    This is what people suggested in the Bark issue on GitHub. ([GitHub](https://github.com/suno-ai/bark/issues/626?utm_source=chatgpt.com "Weight Issue with Pytorch 2.6 Bug Fx #626"))
    
    ### Option B — Keep safe mode, but allow the missing global(s)
    
    Surround the `torch.load(...)` call with a safe globals context, or add safe globals first.
    
    Example:
    
    ```python
    import torch
    import numpy as np
    
    # allow numpy.core.multiarray.scalar
    torch.serialization.add_safe_globals([np.core.multiarray.scalar])
    
    checkpoint = torch.load(ckpt_path, map_location=device, weights_only=True)
    ```
    
    Or using a context manager:
    
    ```python
    import torch
    import numpy as np
    
    with torch.serialization.safe_globals([np.core.multiarray.scalar]):
        checkpoint = torch.load(ckpt_path, map_location=device, weights_only=True)
    ```
    
3. **Ensure that the rest of Bark's code works** with the loaded checkpoint (the structure, state dict, etc.). No further modifications may be required beyond loading.
    

---

## Example patch

Here’s a more concrete patch you could apply in a typical Bark loader file.

```python
import torch
import numpy as np

def load_bark_checkpoint(ckpt_path, device):
    # Option B: safer approach
    torch.serialization.add_safe_globals([np.core.multiarray.scalar])
    checkpoint = torch.load(ckpt_path, map_location=device, weights_only=True)
    return checkpoint
```

If that fails, switch to Option A:

```python
def load_bark_checkpoint(ckpt_path, device):
    # Less safe but loads full checkpoint
    checkpoint = torch.load(ckpt_path, map_location=device, weights_only=False)
    return checkpoint
```

---

## What Bark maintainers say

According to the Bark GitHub issue, they agree that changing the `torch.load(...)` call to include `weights_only=False` is a valid fix. ([GitHub](https://github.com/suno-ai/bark/issues/626?utm_source=chatgpt.com "Weight Issue with Pytorch 2.6 Bug Fx #626"))

---

```python
import numpy
import torch.serialization

torch.serialization.add_safe_globals([
    (numpy._core.multiarray.scalar, 'numpy.core.multiarray.scalar'),
    numpy.dtype,
    numpy.dtypes.Float64DType
])
```

