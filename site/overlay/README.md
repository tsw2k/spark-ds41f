# Site overlay

Files the profile mounts read-only over the image's source tree
(`container.mounts` in `config/cluster.json`). Each one is the image's file with one change;
remove the mount once the image carries the fix.

## `vllm/vllm/models/deepseek_v4_1/nvidia/model_state.py`

Base: the file in image `spark-ds41f:19f2c20ed4d6-next-6`
(sha256 `b40ea4f61ff3ce2ec26bdaee569109c5b016d7e825dc96c0ed01f170f6fb9966`).
Valid for that image only: a rebuilt image needs a fresh copy or no overlay.

Change: `_early_engram_ids` returns `None` instead of raising when a verifying request
follows a prefill chunk. The step then goes without early Engram IDs and the GPU hashes
every row, as before patch 0148's early reads, so its results are unchanged and only that
step loses the early read.

Why: under mixed load (four agents resending 50K-300K contexts beside eight short chats,
MTP drafts on, 8,192 batched tokens) the scheduler builds such steps. The raise killed the
engine within minutes: 16 times on 2026-10-29, every log ending in

    RuntimeError: Engram host hash: a verifying request follows a prefill chunk, so the
    chunk's rows are placed by the GPU
