# Bash Commands Guide for Beginners (Bioinformatics Context)

This tutorial covers essential bash commands with bioinformatics examples.
Open a terminal and run the commands as you read.

## Table of Contents

1. [Navigating files: pwd, ls, cd](#1-navigating-files)
2. [Looking at files: cat, less, head, tail](#2-looking-at-files)
3. [Creating stuff: mkdir, touch, echo, >](#3-creating-stuff)
4. [Copy / Move / Delete: cp, mv, rm](#4-copy--move--delete)
5. [File permissions: chmod, chown, ls -l](#5-file-permissions--ownership)
6. [Counting & sorting: wc, sort, uniq](#6-counting--sorting)
7. [Finding patterns: grep](#7-finding-patterns)
8. [Streaming: | (pipe)](#8-streaming)
9. [Wildcards: \* ? \[ \]](#9-wildcards)
10. [Variables: name="value", $name](#10-variables)
11. [Loops: for, while](#11-loops)
12. [Conditionals: if, test](#12-conditionals)
13. [Scripts & shebang](#13-scripts--shebang)
14. [Real mini pipeline](#14-real-mini-pipeline)

---

## 1. Navigating files

### `pwd` — print working directory

Shows where you are in the filesystem right now. Run it anytime you feel lost.

```bash
pwd
```

Output (yours will be different):

```
/home/giangnguyen/Documents/dev/bioinformatics-4-all
```

### `ls` — list directory contents

Shows files and folders in the current directory.

```bash
ls
```

Output:

```
bash  README.md
```

```bash
ls -l
```

`-l` = "long format" — shows permissions, owner, group, size, date.

Output:

```
drwxrwxr-x 2 giangnguyen giangnguyen 4096 Jun 24 12:00 bash
-rw-rw-r-- 1 giangnguyen giangnguyen   24 Jun 24 12:00 README.md
```

Breakdown of the first column (mode string):

```
d rwx rwx r-x
│ └┬─ └┬─ └┬─
│  │   │   └── permissions for "others" (everyone else)
│  │   └────── permissions for "group" (giangnguyen)
│  └────────── permissions for "user/owner" (giangnguyen)
└───────────── file type: d=directory, -=regular file
```

Each triplet: `r` = read, `w` = write, `x` = execute, `-` = no permission.

The two columns after the mode are the **owner** and **group**:

```
drwxrwxr-x 2 giangnguyen giangnguyen ...
               └─owner──┘ └─group──┘
```

```bash
ls -lh
```

`-h` = "human readable" — sizes like `4.0K` instead of `4096`.

```bash
ls -la
```

`-a` = "all" — also shows hidden files (names starting with dot).

```bash
ls bash/
```

List contents of a specific directory.

### `cd` — change directory

Move into a different folder.

```bash
cd bash          # go into the bash/ folder
pwd              # now you're in .../bioinformatics-4-all/bash
cd ..            # go back up one level (.. = parent directory)
pwd              # back to .../bioinformatics-4-all
cd               # cd with no arguments goes to your home directory
cd -             # go back to the previous directory
cd /tmp          # absolute path — go to /tmp no matter where you are
```

In bioinformatics you often navigate between project folders:

```bash
cd ~/projects/rnaseq/data/raw/
```

---

## 2. Looking at files

### `cat` — print entire file to screen

"concatenate" — dumps everything. Good for small files only.

```bash
cat README.md
```

### `less` — view file page by page

Press:

- `Space` / `f` → next page
- `b` → previous page
- `/word` → search for "word"
- `q` → quit

```bash
less README.md
```

### `head` — show the first 10 lines

Perfect for peeking at FASTQ, BED, or VCF files without loading the whole thing.

```bash
head README.md
head -n 5 README.md        # show first 5 lines instead of 10
```

In bioinformatics you use `head` to check the format of a FASTQ file:

```bash
# head -n 8 sample.fastq.gz   (won't work on .gz — use zcat first)
```

### `tail` — show the last 10 lines

```bash
tail README.md
tail -n 5 README.md
```

Useful for checking that a file ends correctly (e.g., VCF header vs data):

```bash
# tail -n 3 sample.vcf
```

---

## 3. Creating stuff

### `mkdir` — make a new directory

```bash
mkdir test_folder
ls -d test_folder/     # show the directory
```

```bash
mkdir -p data/samples/control
```

`-p` = "parents" — creates intermediate folders as needed.
This creates `data/`, then `data/samples/`, then `data/samples/control/`.
Without `-p` it would fail if `data/` doesn't exist.

### `touch` — create an empty file (or update its timestamp)

```bash
touch empty_file.txt
ls -l empty_file.txt
```

If the file already exists, `touch` just updates its "last modified" time.

### `echo` — print text to the screen

```bash
echo "Hello, bioinformatics"
echo "Sample: SRR12345678"
```

### `>` and `>>` — redirect output to a file

- `>` → write output to file (overwrites)
- `>>` → append output to file (adds to end)

```bash
echo ">read1" > test.fasta          # create test.fasta with one line
echo "ACGTACGTACGT" >> test.fasta   # append a second line
cat test.fasta
```

In bioinformatics you redirect tool output:

```bash
# bwa mem ref.fa reads.fq > aligned.sam
# samtools sort aligned.bam -o sorted.bam       # -o is the output flag
# fastqc reads.fq -o qc_reports/                 # -o specifies output dir
```

---

## 4. Copy / Move / Delete

### `cp` — copy files and directories

```bash
cp test.fasta test_copy.fasta
ls test_copy.fasta

cp -r data/ data_backup/
```

`-r` = "recursive" — needed for copying directories.

### `mv` — move (or rename) files

```bash
mv test_copy.fasta renamed.fasta
ls test_copy.fasta          # error — file no longer exists
ls renamed.fasta            # found here

mv renamed.fasta data/      # move into data/ folder
ls data/
```

### `rm` — remove (delete) files

> **⚠️ WARNING:** There is NO trash bin. Deleted files are gone forever.

```bash
rm test.fasta
rm data/renamed.fasta

rm -r test_folder data_backup data/
```

`-r` = "recursive" — required for directories (deletes everything inside).
Think 10 times before running `rm -rf` on anything important.

```bash
rm -f junk.txt
```

`-f` = "force" — doesn't ask for confirmation, doesn't error if file missing.

When cleaning up intermediate files, use `rm` with explicit filenames:

```bash
# rm -f *.sam  *.bam  *.fastq   (but only after confirming you're done)
```

---

## 5. File Permissions & Ownership

### Permission modes (symbolic)

Every file and directory has three permission triples:

```
owner (u)    group (g)    others (o)
 rwx          rwx          rwx
```

`r` = read (4), `w` = write (2), `x` = execute (1).

Examples from `ls -l`:

```
-rwxr-xr-x   → owner can rwx, group can rx, others can rx
-rw-------   → owner can rw, nobody else can do anything
drwxr-x---   → directory, owner can rwx, group can rx, others nothing
```

### Permission modes (numeric / octal)

Each triple is a sum of its bits:

| # | Sum  | Mode  |
|---|------|-------|
| 7 | 4+2+1 | rwx |
| 6 | 4+2+0 | rw- |
| 5 | 4+0+1 | r-x |
| 4 | 4+0+0 | r-- |
| 3 | 0+2+1 | -wx |
| 2 | 0+2+0 | -w- |
| 1 | 0+0+1 | --x |
| 0 | 0+0+0 | --- |

A three-digit number sets owner / group / others at once:

```
777  =  rwxrwxrwx   (everyone can do everything — dangerous)
755  =  rwxr-xr-x   (common for directories & scripts)
700  =  rwx------   (only owner can access — good for private keys)
644  =  rw-r--r--   (common for data files)
600  =  rw-------   (private data file)
```

Formula: `chmod <owner><group><others> <file>`

### `chmod` — change file mode (permissions)

```bash
echo "private data" > secret.txt
ls -l secret.txt
```

```
-rw-rw-r--  ...   ← created with default permissions (usually 664)
```

```bash
chmod 600 secret.txt
ls -l secret.txt
```

```
-rw-------  ...   ← now only owner can read/write
```

You can also use symbolic notation:

```bash
chmod u+x secret.txt    # add execute for user
chmod go-r secret.txt   # remove read for group and others
chmod a+r secret.txt    # add read for all (a = user+group+others)
```

Most common bioinformatics use:

```bash
chmod +x run.sh         # make a script executable

rm secret.txt
```

### `chown` — change file owner and/or group

Only root or `sudo` can change ownership.
(These commands will fail if you run them without sudo — shown for reference.)

```bash
# sudo chown alice secret.txt              # change owner to alice
# sudo chown :bio-group secret.txt         # change group to bio-group
# sudo chown alice:bio-group secret.txt    # change both
```

### When permissions matter in bioinformatics

- Scripts need `+x` (executable) before you can run them with `./script.sh`
- Shared group folders use `775` so lab members can all write to them
- Private key files (`.pem`, `.key`) MUST be `600` or ssh will refuse them
- Database or reference files are often `644` (readable by everyone)
- Use `ls -l` to check if a file is readable before running a tool on it

---

## 6. Counting & sorting

### `wc` — word count (lines, words, characters)

```bash
wc README.md
```

```
 4  12  73 README.md
 lines words chars filename
```

```bash
wc -l README.md
```

`-l` = lines only → `4 README.md`.

In bioinformatics:

```
wc -l tells you how many records a file has.
For BED files, each line is one region.
For VCF files, each line (after headers) is one variant.
For FASTQ files, count ÷ 4 = number of reads.
```

### `sort` — sort lines alphabetically / numerically

```bash
echo -e "chr3\nchr1\nchr10\nchr2" | sort
```

```
chr1  chr10  chr2  chr3
```

Note: "chr10" comes before "chr2" because it's alphabetical, not numeric.

```bash
echo -e "chr3\nchr1\nchr10\nchr2" | sort -V
```

`-V` = "version sort" — understands numbers properly.

```
chr1  chr2  chr3  chr10
```

### `uniq` — remove / count duplicate adjacent lines

> **IMPORTANT:** `uniq` only removes duplicates that are NEXT to each other.
> Always `sort` before `uniq`.

```bash
echo -e "A\nB\nA\nB\nB" | sort | uniq
```

```
A  B
```

```bash
echo -e "A\nB\nA\nB\nB" | sort | uniq -c
```

`-c` = "count" — shows how many times each line appeared.

```
2 A    3 B
```

In bioinformatics:

```bash
# cut -f 1 annotations.bed | sort | uniq -c | sort -rn
# (counts how many regions per chromosome)
```

---

## 7. Finding patterns

### `grep` — print lines matching a pattern

The single most useful command in bioinformatics.

```bash
echo -e ">chr1\nACGT\n>chr2\nTGCA" > sequences.fa

grep ">" sequences.fa
```

```
>chr1  >chr2
```

(prints only the header lines in a FASTA file)

```bash
grep -c ">" sequences.fa
```

`-c` = "count" — just prints the number of matching lines.

```bash
grep -v ">" sequences.fa
```

`-v` = "invert" — print NON-matching lines (only sequences, not headers).

```bash
grep -i "chr1" sequences.fa
```

`-i` = "case insensitive".

```bash
grep -w "gene" annotation.gtf
```

`-w` = "whole word" — only match "gene", not "genes" or "pseudogene".

Bioinformatics power combo:

```bash
# grep -v "^#" input.vcf | wc -l
#   (count variants in a VCF — skip comment lines that start with #)

rm sequences.fa
```

### `zgrep` — grep inside gzipped files (no need to decompress)

```bash
# zgrep ">" file.fasta.gz
# zgrep -c "PASS" file.vcf.gz
```

---

## 8. Streaming

### `|` — send output of one command into the next command

This is the superpower of bash. You chain small tools together.

```bash
ls -l | wc -l
```

Count how many files are in the current directory.

```bash
head -n 100 big_file.txt | tail -n 20
```

Show lines 81-100: first take first 100 lines, then take last 20 of those.

Common bioinformatics pipe pattern:

```bash
# zcat reads.fastq.gz | head -n 4
#   (peek at first 4 lines of a zipped FASTQ = 1 read)
```

Another:

```bash
# cat sample.bed | wc -l
#   (count regions — unnecessarily using cat, same as wc -l sample.bed)
```

The "useless use of cat" award:

```bash
# cat file | grep "pattern"     ← useless, just grep "pattern" file
# grep "pattern" file            ← correct, fewer processes
```

---

## 9. Wildcards

### `*` — matches ANY sequence of characters

```bash
ls *.sh          # all files ending in .sh
ls data_*        # all files starting with "data_"
ls *_R1.fastq.gz # all forward read FASTQ files
```

### `?` — matches exactly ONE character

```bash
ls sample_?.fastq
```

Matches `sample_1.fastq`, `sample_2.fastq` ... but NOT `sample_10.fastq`.

### `[ ]` — matches ONE character from a set

```bash
ls sample_[123].fastq
```

Matches `sample_1.fastq`, `sample_2.fastq`, `sample_3.fastq`.

```bash
ls sample_[1-3].fastq
```

Same as above — `[1-3]` = range from 1 to 3.

In bioinformatics you loop over wildcards:

```bash
# for f in *.fastq.gz; do echo "Processing $f"; done
```

---

## 10. Variables

Variables store values so you don't retype them.

**Rules:**

- NO spaces around `=` — `name = "value"` is WRONG
- Use `$name` or `${name}` to read the value
- Quote `"$variable"` to handle spaces correctly

```bash
sample="SRR12345678"
echo $sample

input="${sample}_R1.fastq.gz"
echo $input
```

```
SRR12345678_R1.fastq.gz
```

Now you can use the variable in commands:

```bash
# fastqc "$input" -o qc_reports/
```

### Special variables

```bash
echo "Script name:  $0"       # ./common.sh  (or whatever you called it)
echo "First argument: $1"     # first word after script name
echo "Second argument: $2"    # second word
echo "All arguments:  $*"     # everything
echo "Number of args: $#"     # count
```

### Command substitution: `$(command)`

Store output of a command in a variable:

```bash
num_lines=$(wc -l < README.md)
echo "README has $num_lines lines"
```

In bioinformatics:

```bash
# sample_name=$(basename "$fastq_file" .fastq.gz)
# echo "Processing $sample_name"
```

---

## 11. Loops

### `for` — repeat for each item in a list

```bash
echo "ATCGTAGCTAGCTAG" > seq1.fa
echo "GGCTAGCTAGCTAGA" > seq2.fa
echo "TTTAGCTAGCTAGCT" > seq3.fa

for file in seq1.fa seq2.fa seq3.fa; do
    echo "File: $file"
    wc -c < "$file"
done
```

Loop over all `.fa` files with wildcards:

```bash
for file in *.fa; do
    echo "Processing $file"
    # grep ">" "$file"   # get header lines
done
```

In bioinformatics you loop over samples:

```bash
# for sample in SRR1234567 SRR1234568 SRR1234569; do
#     fastqc "${sample}_R1.fastq.gz" -o qc_reports/
# done
```

### `while` — repeat while a condition is true

Read a file line by line:

```bash
while read -r line; do
    echo "Line: $line"
done < README.md
```

In bioinformatics you might iterate over a sample sheet:

```bash
# while IFS=',' read -r sample_id condition; do
#     echo "Processing $sample_id ($condition)"
# done < samples.csv

rm seq1.fa seq2.fa seq3.fa
```

---

## 12. Conditionals

### `if` / `then` / `else` / `fi` — make decisions in your script

Check if a file exists:

```bash
file="README.md"
if [[ -f "$file" ]]; then
    echo "$file exists"
else
    echo "$file does not exist"
fi
```

Check number of arguments:

```bash
# if [[ $# -eq 0 ]]; then
#     echo "Usage: $0 <input.fastq>"
#     exit 1
# fi
```

### Common test operators

| Operator | What it checks |
|----------|---------------|
| `-f "$file"` | file exists |
| `-d "$dir"` | directory exists |
| `-s "$file"` | file exists AND is non-empty |
| `-z "$string"` | string is empty (zero length) |
| `-n "$string"` | string is non-empty |
| `"$a" == "$b"` | strings equal |
| `"$a" != "$b"` | strings not equal |
| `"$n1" -gt "$n2"` | number1 greater than number2 |
| `"$n1" -eq "$n2"` | numbers equal |

### One-liner with `&&` and `||`

- `&&` = "and" — run next command only if previous succeeded
- `||` = "or" — run next command only if previous failed

```bash
[[ -f "README.md" ]] && echo "Found it"
[[ -f "NONEXISTENT" ]] || echo "Not found"
```

In bioinformatics:

```bash
# [[ -f "$input" ]] || die "Input file $input not found"
```

---

## 13. Scripts & shebang

A bash script is just a text file with commands, plus a shebang at the top.

1. Create a file called `run.sh`:

```bash
echo '#!/usr/bin/bash' > run.sh
echo 'echo "Hello from script"' >> run.sh
```

2. Make it executable:

```bash
chmod +x run.sh
```

3. Run it:

```bash
./run.sh
```

The shebang (`#!`) tells the system which interpreter to use.
`/usr/bin/bash` is more portable than `/bin/bash`.

Always include these at the top of every script:

```bash
#!/usr/bin/bash
set -euo pipefail
```

- `set -e` → exit on error
- `set -u` → error on undefined variable
- `set -o pipefail` → fail if any command in a pipe fails

---

## Generate example FASTQ files

Before running the mini pipeline below, create a sample FASTQ file with 4 reads:

```bash
cat > sample.fastq <<'EOF'
@SRR1234567.1 read1 length=36
ACGTACGTACGTACGTACGTACGTACGTACGTACGT
+
FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF
@SRR1234567.2 read2 length=36
TGCAACGTACGTACGTACGTACGTACGTACGTACGT
+
FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF
@SRR1234567.3 read3 length=36
GGCTAGCTAGCTAGCTAGCTAGCTAGCTAGCTAGCT
+
FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF
@SRR1234567.4 read4 length=36
CCCTAGCTAGCTAGCTAGCTAGCTAGCTAGCTAGCT
+
FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF
EOF

gzip sample.fastq            # create sample.fastq.gz
rm sample.fastq              # remove the uncompressed original
ls -lh sample.fastq.gz       # verify it exists
```

---

## 14. Real mini pipeline

Below is a complete, working bioinformatics script.
It counts how many reads are in a FASTQ file and extracts the headers.

First run the [FASTQ generation commands](#generate-example-fastq-files) above, then:

```bash
bash count_reads.sh sample.fastq.gz
```

Save this script as `count_reads.sh`:

```bash
#!/usr/bin/bash
# count_reads.sh — count reads in a FASTQ file and show first 5 headers
set -euo pipefail

# --- Check arguments ---
if [[ $# -eq 0 ]]; then
    echo "Usage: $0 <input.fastq.gz>"
    exit 1
fi

input="$1"

# --- Check file ---
if [[ ! -f "$input" ]]; then
    echo "ERROR: File '$input' not found"
    exit 1
fi

echo "========================================"
echo "  FASTQ Reader"
echo "========================================"

# --- Count lines ---
if [[ "$input" == *.gz ]]; then
    total_lines=$(zcat "$input" | wc -l)
else
    total_lines=$(wc -l < "$input")
fi

total_reads=$(( total_lines / 4 ))
echo "  File:      $input"
echo "  Lines:     $total_lines"
echo "  Reads:     $total_reads"
echo ""

# --- Show first 5 read headers ---
echo "First 5 read headers:"
if [[ "$input" == *.gz ]]; then
    zcat "$input" | head -n 20 | grep "^@"
else
    head -n 20 "$input" | grep "^@"
fi

echo ""
echo "Done!"
```

---

## What you learned

| Command | What it does | Bioinfo use |
|---------|-------------|-------------|
| `pwd` | Where am I? | Find your project folder |
| `ls` | What files are here? | Check for FASTQ / BAM files |
| `ls -l` | Show owner, group, permissions | Verify file access, ownership |
| `chmod` | Change file permissions | Make script executable (+x) |
| `chown` | Change file owner | Fix ownership after copy/move |
| `cd` | Go somewhere | Navigate data directories |
| `cat` | Print whole file | Peek at small files |
| `less` | Browse file page by page | Inspect huge VCF / BED files |
| `head`/`tail` | First / last lines | Check FASTQ header, file end |
| `mkdir` | Create folder | Organize outputs by step |
| `touch` | Create empty file | Placeholder / marker files |
| `echo` | Print text | Status messages in scripts |
| `>` / `>>` | Redirect to file | Save tool output |
| `cp` | Copy | Back up raw data |
| `mv` | Move / rename | Organize results |
| `rm` | Delete (⚠️ permanent!) | Clean up intermediates |
| `wc` | Count lines / words / chars | Count reads, variants, regions |
| `sort` | Sort lines | Order genomic coordinates |
| `uniq` | Find / count unique lines | Deduplicate, count features |
| `grep` | Find matching lines | Extract genes, filter VCF |
| `\|` | Pipe (chain commands) | Build pipelines from tools |
| `*` `?` `[ ]` | Wildcards | Batch-process all FASTQ files |
| `$var` | Variables | Store sample names, paths |
| `for`/`while` | Loops | Process multiple samples |
| `if`/`[[ ]]` | Conditionals | Check files, validate inputs |
