# ansible-sar

A playbook to scan and rewrite ansible projects using Steampunk Spotter. 

The playbook expects the following variables:
- `working_dir` the working directory
- `csv` the CSV file

The tarball archives for each CSV file should be placed in a folder named after the CSV file (without the extension) under `$WORKING_DIR/archives/` folder.

Example

```sh
➜  ansible-sar: tree -L 3 out
out                                 # Working director
├── archives
│   └── raw                         # Archives for raw.csv
│       ├── ad-hoc-playbooks.tar
│       ├── ans-admin.tar
│       ├── apm-activegate.tar.gz
│       └── win-agent-ansible.tar
└── raw.csv                         # The CSV files
```

The playbook will:
- Validate the inputs
- Compute the following
    - list of repos that can be processed
    - list of repos missing the archive (tar ball)
    - list of repos with multiple top level directories (skipped)
- Delete and re-create the extracted folder if needed
- Process each repo
    - Extract the archive in `$WORKING_DIR/extracted/<CSV FILE NAME WITHOUT EXTENSION>`
    - Scan and rewrite the repo
    - Re-scan after rewriting
    - Save the scan result in the `___spotter-report___.txt` file located in the extracted repo.
- Print a summary (processed, skipped, etc.)


## Setup

1. Create the working and archives directories

```sh
WORKING_DIR=./out
mkdir -p $WORKING_DIR/archives
```

2. Download the CSV (spreadsheet) in $WORKING_DIR

```sh
cp .../raw.csv $$WORKING_DIR/raw.csv
```

3. Create a folder and download the repo archives (tar balls) for the CSV file

Note: The folder must be named same as the CSV file minus the extension

```sh
### 1. Create the archives folder for the CSV
mkdir $WORKING_DIR/archives/raw

### 2. Download the archives (tar balls) in the folder
```

4. Install the playbook dependencies

```sh
ansible-galaxy collection install -r requirements.yml -p ./collections
```

5. Run the playbook

```sh
# Set the environment variables
export SPOTTER_ENDPOINT=...
export SPOTTER_TOKEN=...

# Run the playbook
ansible-playbook playbook.yml --extra-vars "working_dir=$WORKING_DIR csv=$WORKING_DIR/raw.csv" --ask-become-pass
```

Notes:
- `--ask-become-pass` is needed to delete extracted files if needed
- `--skip-tags spotter` can be used to skip spotter related tasks.
