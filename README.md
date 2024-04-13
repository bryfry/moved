
## TODO

- [ ] blog.alta3.com needs tags or catigories
  - currently using 302 for:
  - /blog/tag/* -> blog.alta3.com
  - /blog/category/* -> blog.alta3.com

#### Backup 

```bash
sqlite3 moved.db '.dump redir' > data.sql
```

#### caddy fmt output

```bash
sqlite3 moved.db \
    '.mode json' \
    'select * FROM redir' \
    | jq -j -r '
      .[] 
      | "redir ", .src, .dst, .status, "\n"'
```
