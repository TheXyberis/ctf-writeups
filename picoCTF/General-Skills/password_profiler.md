# Password Profiler — picoCTF 2026

* **Platform:** picoCTF 2026
* **Category:** General Skills
* **Difficulty:** Easy
* **Author:** Yahaya Meddy

---

## 1. Challenge Overview

We intercepted a suspicious file from a system, but instead of the password itself, it only contains its SHA-1 hash. Using OSINT techniques, you are provided with personal details about the target. Your task is to leverage this information to generate a custom password list and recover the original password by matching its hash. Download the following files:

Provided files:
* `userinfo.txt` — Personal details about the target.
* `hash.txt` — Target SHA-1 hash (`3d92bc3071ba0fb04cf57eaabc3d23135dbce2d0`).
* `check_password.py` — Python script that reads candidate passwords from `passwords.txt`, computes their SHA-1 hashes, and checks for a match against `hash.txt`.

---

## 2. Initial Inspection

First, I inspect the personal information in `userinfo.txt`, the target hash in `hash.txt`, and the verification logic in `check_password.py`:

```text
First Name: Alice
Surname: Johnson
Nickname: AJ
Birthdate: 15-07-1990
Partner's Name: Bob
Child's Name: Charlie
```

```text
3d92bc3071ba0fb04cf57eaabc3d23135dbce2d0
```

```python
#!/usr/bin/env python3
import hashlib

HASH_FILE = "hash.txt"
WORDLIST_FILE = "passwords.txt"  # wordlist that was generated using CUPP


def load_hash():
    with open(HASH_FILE, "r") as f:
        return f.read().strip()


def crack_password(target_hash):
    with open(WORDLIST_FILE, "r", encoding="utf-8", errors="ignore") as f:
        for password in f:
            password = password.strip()
            if hashlib.sha1(password.encode()).hexdigest() == target_hash:
                return password
    return None


if __name__ == "__main__":
    target_hash = load_hash()
    result = crack_password(target_hash)
    if result:
        print(f"Password found: academy{{{result}}}")
    else:
        print("No match found.")
```

`check_password.py` expects candidate passwords in `passwords.txt` and prints `academy{<password>}` as soon as a password's SHA-1 hex digest matches `hash.txt`.

---

## 3. Step-by-Step Solution

### Step 1: Generate a Custom Wordlist with CUPP

Using `cupp.py` (Common User Passwords Profiler) in interactive mode (`-i`), I enter the target's profile details from `userinfo.txt` (formatting the birthdate as `15071990` for `DDMMYYYY`) and enable special characters, random numbers, and leet mode:

```bash
python3 cupp.py -i
```

```text
> First Name: Alice
> Surname: Johnson
> Nickname: AJ
> Birthdate (DDMMYYYY): 15071990

> Partners) name: Bob
> Partners) nickname:
> Partners) birthdate (DDMMYYYY):

> Child's name: Charlie
> Child's nickname:
> Child's birthdate (DDMMYYYY):

> Pet's name:
> Company name:

> Do you want to add some key words about the victim? Y/[N]: n
> Do you want to add special chars at the end of words? Y/[N]: y
> Do you want to add some random numbers at the end of words? Y/[N]:y
> Leet mode? (i.e. leet = 1337) Y/[N]: y

[+] Now making a dictionary...
[+] Sorting list and removing duplicates...
[+] Saving dictionary to alice.txt, counting 29516 words.
> Hyperspeed Print? (Y/n) : n
```

### Step 2: Rename the Wordlist and Run `check_password.py`

CUPP saves the generated dictionary to `alice.txt` (29,516 words), whereas `check_password.py` looks for `passwords.txt`. I rename `alice.txt` to `passwords.txt` and execute the script:

```bash
mv alice.txt passwords.txt
python3 check_password.py
```

**Output:**

```text
Password found: academy{aliceaj7515}
```

---

## 4. Flag

```text
academy{aliceaj7515}
```
