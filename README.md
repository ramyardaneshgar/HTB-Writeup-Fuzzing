# Fuzzing
Advanced Recursive Fuzzing with ffuf to Uncover Hidden Web Directories, Files, and Configuration Parameters.

Developed by Ramyar Daneshgar

## Objective
The lab involved identifying hidden directories, files, and other sensitive content within a web application using recursive fuzzing techniques. 

---

## Steps Taken

### 1. **Initial Setup**
To begin, I ensured that the target IP address could be resolved correctly. Since the target application didn’t require a specific hostname, this step wasn’t necessary for this case, but it’s critical when dealing with virtual hosts. If needed, I would map the target IP to a hostname in `/etc/hosts`:

```bash
sudo sh -c 'echo "68.183.45.211 academy.htb" >> /etc/hosts'
```

This step ensures that tools like **Ffuf** correctly resolve the target host during fuzzing.

---

### 2. **Directory Fuzzing**
I performed an initial scan to identify directories at the root of the web application. I used the `directory-list-2.3-small.txt` wordlist from **SecLists**, which balances coverage and performance. 

The command used:
```bash
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://68.183.45.211:32449/FUZZ -v
```

#### Explanation:
- **`-w`**: Specifies the wordlist, with `FUZZ` as the placeholder for each entry.
- **`-u`**: The target URL where `FUZZ` is replaced by each wordlist entry.
- **`-v`**: Enables verbose mode for detailed output.

**Output**:
```plaintext
/forum
/blog
```
This indicated two accessible directories at the root level.

---

### 3. **Recursive Fuzzing**
To identify subdirectories and files within `/forum` and `/blog`, I enabled recursive fuzzing with the `-recursion` flag and limited exploration depth to 1 using `-recursion-depth 1` to avoid excessive requests.

Command:
```bash
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://68.183.45.211:32449/FUZZ -recursion -recursion-depth 1 -v
```

#### Explanation:
- **`-recursion`**: Automatically scans subdirectories found in the initial scan.
- **`-recursion-depth 1`**: Restricts the scan to one level of subdirectories.

**Output**:
```plaintext
/forum/index.php
/forum/flag.php
```
This quickly revealed additional files within the `/forum` directory.

---

### 4. **Fuzzing with Specific Extensions**
To refine the search, I focused on files with the `.php` extension by using the `-e` flag. This targeted common dynamic web application files and reduced noise from other extensions.

Command:
```bash
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://68.183.45.211:32449/FUZZ -recursion -recursion-depth 1 -e .php -v
```

#### Explanation:
- **`-e .php`**: Appends the `.php` extension to each wordlist entry during the scan.

**Output**:
```plaintext
/forum/index.php
/forum/flag.php
```
The scan confirmed the presence of PHP files, including the critical `/forum/flag.php`.

---

### 5. **Flag Verification**
To confirm the result, I accessed the `/forum/flag.php` file directly using **curl**:

```bash
curl http://68.183.45.211:32449/forum/flag.php
```

#### Output:
```plaintext
HTB{fuzz1n6_7h3_w3b!}
```
The content of the file revealed the required flag.

---

## Tools and Commands Used

1. **Ffuf**: For directory and file enumeration.
   - Key options: `-recursion`, `-recursion-depth`, `-e`, and `-v`.
2. **SecLists**: Provided the `directory-list-2.3-small.txt` wordlist.
3. **curl**: Verified the identified paths manually.

---

## Lesson Learned

1. **Recursive Fuzzing**:
   Automating subdirectory exploration with recursion significantly reduced manual effort and time. The `-recursion` flag allowed me to discover nested files efficiently.

2. **Focused Searches**:
   By targeting the `.php` extension, I refined the search results, avoiding irrelevant file types.

3. **Manual Validation**:
   While automation speeds up discovery, manually verifying results ensures accuracy and confirms their relevance.

4. **Wordlist Selection**:
   Using a concise but effective wordlist like `directory-list-2.3-small.txt` was critical for meaningful results within a reasonable timeframe.


