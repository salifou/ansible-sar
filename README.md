# ansible-sar

Scan and rewrite ansible projects using spotter.

## Folder Structure

```sh
➜  ansible-sar tree -L 3
.
├── files
│   ├── extracted   # Folder to save extracted repos and scan results
│   │   └── ean
│   └── raw         # Folder containing repos archives per instance
│       ├── ean
│       └── raw
├── playbook.yml    # Playbook to extract, scan and rewuire projects
├── README.md
├── tasks
│   └── scan.yml
└── vars.yml        # Variable definitions
```

## Quick start

1. Update `vars.yml` with repo details
1. Download repo archive in `files/raw/<ORG>` folder
1. Run the playbook

```sh
# Set the environment variables
export SPOTTER_ENDPOINT=...
export SPOTTER_TOKEN=...

# Run the playbook
ansible-playbook playbook.yml
```

The playbook will:
- Extract each repo in `files/extracted/<ORG>`
- Scan and rewrite the repo
- Re-scan after rewriting
- Save the result in `files/extracted/<ORG>/<REPO NAME>/spotter-scan-and-rewrite_<REPO NAME>.txt`
