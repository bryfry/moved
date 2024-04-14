
## TODO

- Import sources from:
  - [x] squarespace sitemap
  - [x] squarespace redirects
  - [x] google analytics 
  - [x] blog moved files (redirect.yaml)
  - [ ] google search console
- [x] blog.alta3.com needs tags or catigories
  - currently using 302 for:
  - /blog/tag/* -> blog.alta3.com
  - /blog/category/* -> blog.alta3.com
- [x] posters need migrated to blog.alta3.com/posters
- [ ] aws paths need real target paths
- [ ] routes with query parameters
  - /store?tags={search} -> /courses?search={search}
  - /store?category={search} -> 

## References

- https://caddyserver.com/docs/caddyfile/directives/redir
- https://caddyserver.com/docs/caddyfile/directives/import

## Use

#### Backup 

```bash
mkdir -p backup
sqlite3 moved.db '.dump redir' > backup/data.sql
```

#### Test repeatable import

```bash
sqlite3 moved.db < data.sql
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

#### google analytics

```bash
cut -d ',' -f 1 source/ga.csv \
  | tail --lines=+2 \
  | xargs -I {} sqlite3 moved.db \
    'INSERT INTO redir(src) 
     SELECT x FROM (SELECT "{}" AS x) 
     WHERE NOT EXISTS(SELECT src FROM redir WHERE src = "{}");'
```

```bash
# get null values back out
sqlite3 moved.db 'select src from redir where dst is NULL' > no_redirect.txt
```

#### google search

```bash
xargs -a source/google_search.txt -I {} \
  sqlite3 moved.db \
    'INSERT INTO redir(src) 
     SELECT x FROM (SELECT "{}" AS x) 
     WHERE NOT EXISTS(SELECT src FROM redir WHERE src = "{}");'
```

Get null values back out:

```bash
sqlite3 moved.db 'select src from redir where dst is NULL' > no_redirect.txt
```

#### blog sources

```bash
# find the new path inside the blogs dir
yq -r -j '.redirects[] | .old, " ", .new, "\n"' < source/redirects.yaml \
  | xargs -n2 bash -c 'echo -n "$0 "; find . -type f -name $1.md' > blogs.txt
```
