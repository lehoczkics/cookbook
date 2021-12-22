## Replicate an entire dataset with all snapshots to other host

It is advised to perform below steps inside a `screen` session so one can close the terminal.<br>

On the source machine:
```
# Get dataset name (first column)
zfs list

# Create a recursive snapshot from the dataset
zfs snap -r tank/my/dataset@replicate

# Send over the snapshot recursively so all parent snapshots are sent as well to target machine
zfs send -R tank/my/dataset@replicate | ssh whoami@target sudo zfs receive -Fud tank

# After it completes destroy the replicate snapshot
zfs destroy tank/my/dataset@replicate

```

