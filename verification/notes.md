---
description: Some useful tricks
---

# Notes

#### Remove trailing symbols

Copying text across platforms \(e.g., Linux and Windows\) may sometime results in trailing symbols. To resolve the issue:

```bash
sed -e "s/^M//" filename > newfilename
```

