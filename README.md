# ansible-sar

A playbook to scan and rewrite ansible projects using Steampunk Spotter. 

The playbook expects the following variables:
- `working_dir` the working directory
- `csv` the CSV file

The archives (tarballs) for each CSV file should be placed in a folder named after the CSV file (without the extension) under `$WORKING_DIR/archives/` folder.

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
    - list of repos missing spotter project uuid
    - list of repos missing the archive (tar ball)
    - list of repos with multiple top level directories (skipped)
- Delete and re-create the extracted folder
- Process each repo
    - Extract the archive in `$WORKING_DIR/extracted/<CSV FILE NAME WITHOUT EXTENSION>`
    - Scan and rewrite the repo 5 times
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

**Playbook Variables**
- `ansible_core_version`: Ansible core version to scan against. Default: "2.16"
- `rewrite_count`: Spotter rewrite count. Default: 5
- `spotter_extra_flags='--profile full'`: Additional flags to `spotter scan` command. Default: ""

**Notes**
- `--ask-become-pass` is needed to delete extracted files if needed
- `--skip-tags spotter` can be used to skip spotter related tasks.



## Workflow

1. Create a Spotter organization(e.g., per RH case or AWX/AAP organization).
2. Create Spotter projects within the organization
3. Create a spreadsheet including the repository names and the Spotter project UUIDs.
4. Create the working directory and download the repository archives (tar balls).
5. Run the playbook, specifying the working directory and the spreadsheet (CSV).
6. Manually address problematic repositories, specifically:
	- Unexpected archive (tarball) name.
		- Expected tarball name `<name in CSV>.{tar, tgz, tar.gz}`
		- See missing archives in playbook summary.
	- Duplicated repository names in the spreadsheet.
	- Archives including multiple folders or mismatching names.
7. Compress the output and send it to the customer.
