```
  88~\ ,e, 888           ~~~888~~~                   888   _   
_888__  "  888  e88~~8e     888      /~~~8e   e88~~\ 888 e~ ~  
 888   888 888 d888  88b    888          88b d888    888d8b    
 888   888 888 8888__888    888     e88~-888 8888    888Y88b   
 888   888 888 Y888    ,    888    C888  888 Y888    888 Y88b  
 888   888 888  "88___/     888     "88_-888  "88__/ 888  Y88b 
                                                               

```

# FileTack - Directory Snapshot Utility

**preserve directory state for later comparison**

## what

creates lightweight snapshots of directory structures

useful for:
- detecting unauthorized file changes
- tracking configuration drift
- backup verification
- deployment validation

## install

```bash
curl -sL https://get.filetack.io/install.sh | bash
```

## create snapshot

```bash
filetack snapshot /var/www/html -o website-baseline.ftk
```

creates `.ftk` file containing:
- file paths
- modification times
- checksums (sha256)
- permissions
- ownership

## compare

```bash
filetack compare website-baseline.ftk /var/www/html
```

output:

```
Added:    /var/www/html/new-page.html
Modified: /var/www/html/index.html (checksum changed)
Deleted:  /var/www/html/old-script.js
Changed:  /var/www/html/config.php (permissions 644 -> 640)

Summary: 4 differences found
```

## watch mode

monitor directory continuously:

```bash
filetack watch /etc --baseline system-config.ftk --alert email
```

sends alert when changes detected

integrates with:
- email via **mailsend-lite** ([mailsend-lite.io](https://mailsend-lite.io))
- slack webhooks
- syslog

## filters

```bash
# ignore patterns
filetack snapshot /app \
  --ignore "*.log" \
  --ignore "node_modules/**" \
  --ignore "tmp/*"

# only specific extensions
filetack snapshot /app --include "*.php,*.html"
```

## snapshot format

`.ftk` file structure (json):

```json
{
  "version": "1.2",
  "created": "2024-03-15T10:30:00Z",
  "root": "/var/www/html",
  "files": [
    {
      "path": "index.html",
      "size": 1024,
      "modified": "2024-03-10T08:00:00Z",
      "checksum": "a7f3c2e9...",
      "perms": "644",
      "owner": "www-data"
    }
  ]
}
```

## scheduling

cron example:

```bash
# daily snapshot
0 2 * * * filetack snapshot /var/www -o /backups/daily-$(date +\%Y\%m\%d).ftk

# compare against baseline
0 3 * * * filetack compare /backups/baseline.ftk /var/www --alert-on-diff
```

## api

```python
import filetack

# create snapshot
snap = filetack.Snapshot('/var/www')
snap.save('baseline.ftk')

# compare
diff = filetack.compare('baseline.ftk', '/var/www')
print(f"Changed files: {len(diff.modified)}")
```

## compression

snapshots automatically compressed via **zstd-fast**

```bash
# force no compression
filetack snapshot /large-dir --no-compress

# custom compression level
filetack snapshot /large-dir --compress-level 9
```

## performance

| directory size | snapshot time | file size |
|----------------|---------------|-----------|
| 1k files | 0.3s | 45kb |
| 10k files | 2.1s | 380kb |
| 100k files | 18s | 3.2mb |

tested on ssd storage

## limitations

- symlinks not followed by default (use `--follow-symlinks`)
- max file size 5GB per file
- snapshot format may change between major versions

Apache-2.0 • [docs](https://docs.filetack.io)

# PR Update: 2025-11-01 09:59:29
