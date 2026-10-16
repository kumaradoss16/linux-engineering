## Real-World Use Case: Generate a Timestamped System Status Report

An administrator creates a timestamped report containing filesystem disk-usage information using the `df` command and output redirection.

### Step 1: Define the report filename

```bash
REPORT="$HOME/cli-lab/system-status-$(date +%F-%H%M%S).txt"
```

**Explanation:**

- `REPORT` — shell variable that stores the report's full path.
- `$HOME` — user's home directory (`/home/hytrel`).
- `cli-lab/` — directory where the report will be saved.
- `date +%F-%H%M%S` — generates a timestamp using the date (`YYYY-MM-DD`) and time (`HHMMSS`).
- `.txt` — file extension for the text report.

Example filename:

```text
/home/hytrel/cli-lab/system-status-2026-10-16-115827.txt
```

### Step 2: Check the report path with `df`

```bash
df -hT "$REPORT"
```

**Output:**

```text
df: /home/hytrel/cli-lab/system-status-2026-10-16-115827.txt: No such file or directory
```

**Explanation:**

- `df` displays filesystem disk-space usage.
- `-h` displays sizes in human-readable units, such as MB and GB.
- `-T` displays the filesystem type.
- `"$REPORT"` supplies the path stored in the variable.

**Why did the command fail?**

The report file has not been created yet. The `df` command expects the supplied path to exist when it is used this way.

### Step 3: Create the report using output redirection

```bash
df -hT > "$REPORT"
```

**Explanation:**

- `df -hT` generates the filesystem usage report.
- `>` redirects standard output into a file.
- `"$REPORT"` specifies the destination filename.
- If the file does not exist, the shell creates it, provided the parent directory exists and permissions allow it.
- If the file already exists, `>` overwrites its contents.

The `df` command only reads filesystem usage information; it does not modify the filesystems being inspected. The shell's redirection creates or overwrites the report file.

### Step 4: Display the report location

```bash
echo "Saved report to: $REPORT"
```

**Output:**

```text
Saved report to: /home/hytrel/cli-lab/system-status-2026-10-16-115827.txt
```

This confirms the path stored in the `REPORT` variable. It does not independently verify that the report was written successfully.

### Step 5: Read the saved report

```bash
cat ~/cli-lab/system-status-2026-10-16-115827.txt
```

**Illustrative output from the provided lab:**

```text
Filesystem     Type     Size  Used Avail Use% Mounted on
tmpfs          tmpfs    392M  1.5M  391M   1% /run
/dev/sda2      ext4      79G  8.4G   66G  12% /
tmpfs          tmpfs    2.0G     0  2.0G   0% /dev/shm
tmpfs          tmpfs    5.0M  8.0K  5.0M   1% /run/lock
tmpfs          tmpfs    392M  148K  392M   1% /run/user/1000
/dev/sr0       iso9660   51M   51M     0 100% /media/hytrel/VBox_GAs_7.2.20
```

Your output may differ depending on the mounted filesystems and current disk usage.

**Output columns:**

| Column | Meaning |
|---|---|
| `Filesystem` | Device or filesystem being reported |
| `Type` | Filesystem type, such as `ext4` or `tmpfs` |
| `Size` | Total filesystem size |
| `Used` | Space currently used |
| `Avail` | Available space |
| `Use%` | Percentage of space used |
| `Mounted on` | Directory where the filesystem is accessible |

### Important observations

1. The first `df -hT "$REPORT"` command fails because the report file does not yet exist.
2. The `>` operator creates the report file and writes the `df` output into it.
3. The `REPORT` variable stores a timestamped path, helping keep reports from different runs separate when their timestamps differ.
4. The `cat ^C` line indicates that the user interrupted a `cat` command with `Ctrl+C`; it is not part of the report-generation process.
5. The `/dev/sr0` entry showing `100%` usage may be normal for a mounted, read-only ISO image. It does not necessarily mean the root filesystem is full.

### Improved command sequence

The following version checks that the destination directory exists, creates the report, and displays the saved file:

```bash
mkdir -p "$HOME/cli-lab"

REPORT="$HOME/cli-lab/system-status-$(date +%F-%H%M%S).txt"

if df -hT > "$REPORT"; then
    echo "Report created successfully: $REPORT"
    cat "$REPORT"
else
    echo "Error: Failed to generate the report." >&2
fi
```

**Real-world relevance:** System administrators can use this technique to save timestamped filesystem reports for routine server checks, troubleshooting, documentation, and before-and-after comparisons. The report records the filesystem information at the time the command runs; it is not a live-updating report.
