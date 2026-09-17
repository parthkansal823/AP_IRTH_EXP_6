# Practical 9 -- Password Hash

## 1. Aim

To perform hash cracking using a password hash and password policy using
Hashcat and Kali Linux, and identify the original password.

## 2. Objective

To crack the password corresponding to the given password hash.

## 3. Apparatus / Requirements

-   Kali Linux VM
-   Hashcat
-   `rockyou.txt` wordlist
-   Password hash
-   Hashcat rule file

## 4. Scenario

A password hash was found in a domain controller.

### Given Hash

``` text
01F3273F68195C29A1A2365BE7AD2B1AAD469A73
```

A note describing the password policy was also found.

## 5. Password Policy

The password follows these rules:

1.  One hexadecimal digit is prepended to the password.
2.  One hexadecimal digit is appended to the password.
3.  The first letter is capitalised.
4.  An `@` is appended to the end.
5.  Leetspeak substitutions are applied.

### Leetspeak Mapping

  Original     Replacement
  ---------- -------------
  `a`                  `4`
  `b`                  `6`
  `e`                  `3`
  `g`                  `9`
  `i`                  `1`
  `o`                  `0`
  `s`                  `5`
  `t`                  `7`
  `z`                  `2`

Hexadecimal characters are:

``` text
0 1 2 3 4 5 6 7 8 9 a b c d e f
```

Therefore, there are 16 possibilities for the prepended digit and 16
possibilities for the appended digit:

``` text
16 × 16 = 256
```

## 6. Create the Experiment Directory

Open the Kali Linux terminal and run:

``` bash
mkdir -p ~/Desktop/Exp9
cd ~/Desktop/Exp9
```

## 7. Store the Hash in `pass.txt`

Create the hash file:

``` bash
echo "01F3273F68195C29A1A2365BE7AD2B1AAD469A73" > pass.txt
```

Verify it:

``` bash
cat pass.txt
```

Expected output:

``` text
01F3273F68195C29A1A2365BE7AD2B1AAD469A73
```

## 8. Identify the Hash Type

Run:

``` bash
hashid pass.txt
```

The hash is identified as a SHA-1 hash.

For Hashcat, SHA-1 uses mode:

``` text
100
```

Therefore:

``` text
-m 100
```

is used during cracking.

## 9. Prepare the Rockyou Wordlist

Move to the wordlist directory:

``` bash
cd /usr/share/wordlists/
```

If `rockyou.txt.gz` is present, extract it:

``` bash
sudo gzip -dk rockyou.txt.gz
```

Check that the wordlist exists:

``` bash
ls -lh rockyou.txt
```

Return to the experiment directory:

``` bash
cd ~/Desktop/Exp9
```

## 10. Create the Hashcat Rule File

Create the rule file:

``` bash
touch pass.rule
```

The common rules are:

``` text
c sa4 sb6 se3 sg9 si1 so0 ss5 st7 sz2
```

### Meaning of the Rules

  Rule    Meaning
  ------- -----------------------------
  `c`     Capitalise the first letter
  `sa4`   Replace `a` with `4`
  `sb6`   Replace `b` with `6`
  `se3`   Replace `e` with `3`
  `sg9`   Replace `g` with `9`
  `si1`   Replace `i` with `1`
  `so0`   Replace `o` with `0`
  `ss5`   Replace `s` with `5`
  `st7`   Replace `t` with `7`
  `sz2`   Replace `z` with `2`
  `$x`    Append character `x`
  `^x`    Prepend character `x`
  `$@`    Append `@`

## 11. Generate All 256 Rules Using Python

Instead of manually writing 256 rules, use Python:

``` bash
python3 -c 'digits="0123456789abcdef"; base="c sa4 sb6 se3 sg9 si1 so0 ss5 st7 sz2"; f=open("pass.rule","w"); [f.write(base+" $"+j+"$@ ^"+i+"\n") for i in digits for j in digits]; f.close()'
```

Check the number of generated rules:

``` bash
wc -l pass.rule
```

Expected:

``` text
256 pass.rule
```

## 12. Python Code Explanation

The same logic can be written more clearly as:

``` python
digits = "0123456789abcdef"

base = "c sa4 sb6 se3 sg9 si1 so0 ss5 st7 sz2"

file = open("pass.rule", "w")

for i in digits:
    for j in digits:
        rule = base + " $" + j + "$@ ^" + i
        file.write(rule + "\n")

file.close()
```

### Line-by-Line Explanation

#### Line 1

``` python
digits = "0123456789abcdef"
```

Stores all 16 hexadecimal characters.

#### Line 3

``` python
base = "c sa4 sb6 se3 sg9 si1 so0 ss5 st7 sz2"
```

Stores the common password transformations that are applied to every
candidate.

#### Line 5

``` python
file = open("pass.rule", "w")
```

Creates `pass.rule` in write mode.

#### Line 7

``` python
for i in digits:
```

Selects the hexadecimal digit that will be prepended.

#### Line 8

``` python
for j in digits:
```

Selects the hexadecimal digit that will be appended.

#### Line 9

``` python
rule = base + " $" + j + "$@ ^" + i
```

Joins the strings together to create one Hashcat rule.

For example, if:

``` text
i = a
j = 5
```

the generated rule is:

``` text
c sa4 sb6 se3 sg9 si1 so0 ss5 st7 sz2 $5$@ ^a
```

Here:

``` text
^a  → prepend a
$5  → append 5
$@  → append @
```

#### Line 10

``` python
file.write(rule + "\n")
```

Writes the rule into the file and moves to the next line.

#### Line 12

``` python
file.close()
```

Closes the rule file.

## 13. Verify the Files

The experiment directory should contain:

``` text
Exp9/
├── pass.txt
└── pass.rule
```

Run:

``` bash
cd ~/Desktop/Exp9
ls -lh
```

Check the hash:

``` bash
cat pass.txt
```

Check the generated rules:

``` bash
head pass.rule
```

Check the total number:

``` bash
wc -l pass.rule
```

## 14. Run Hashcat

Run:

``` bash
sudo hashcat -m 100 pass.txt /usr/share/wordlists/rockyou.txt -r pass.rule
```

If required in the lab environment:

``` bash
sudo hashcat -m 100 pass.txt --force /usr/share/wordlists/rockyou.txt -r pass.rule
```

### Command Explanation

``` text
hashcat
```

Hash cracking tool.

``` text
-m 100
```

Selects SHA-1 hash mode.

``` text
pass.txt
```

Contains the target hash.

``` text
/usr/share/wordlists/rockyou.txt
```

Dictionary containing candidate words.

``` text
-r pass.rule
```

Applies the generated password-policy rules.

## 15. Display the Cracked Password

After Hashcat completes, use:

``` bash
hashcat -m 100 pass.txt --show
```

The recovered password is:

``` text
dCh47rum56@
```

## 16. Reverse the Password Policy

Recovered password:

``` text
dCh47rum56@
```

### Step 1 -- Remove the prepended hexadecimal digit

``` text
dCh47rum56@
↓
Ch47rum56@
```

### Step 2 -- Remove the appended hexadecimal digit

``` text
Ch47rum56@
↓
Ch47rum5@
```

### Step 3 -- Remove `@`

``` text
Ch47rum5@
↓
Ch47rum5
```

### Step 4 -- Reverse leetspeak

``` text
4 → a
7 → t
5 → s
```

Therefore:

``` text
Ch47rum5
↓
Chatrums
```

### Step 5 -- Reverse capitalisation

``` text
Chatrums
↓
chatrums
```

Thus, the underlying word is:

``` text
chatrums
```

## 17. Observations

-   The supplied hash was identified as SHA-1.
-   The password policy required hexadecimal characters at the beginning
    and end.
-   The hexadecimal character set contains 16 possible characters.
-   Two hexadecimal positions produce 256 combinations.
-   A Hashcat rule file was generated automatically using Python.
-   The `rockyou.txt` wordlist was used as the candidate dictionary.
-   Hashcat successfully found a matching password.

## 18. Result

The given SHA-1 hash was successfully cracked using the specified
wordlist and password-policy rules.

### Recovered Password

``` text
dCh47rum56@
```

### Underlying Word

``` text
chatrums
```

## 19. Conclusion

The experiment demonstrates how password-policy information can be
incorporated into Hashcat rules to generate candidate passwords. The
generated rules covered all 256 combinations of the two hexadecimal
positions, and Hashcat was able to identify the password corresponding
to the supplied SHA-1 hash.

## 20. Viva Voce Questions

### Q1. Passwords need to be kept encrypted to protect from offline attacks. True or False?

**Answer:** False. Passwords are generally stored using secure password
hashing rather than reversible encryption.

### Q2. What is the disadvantage of an active online password attack?

An online attack interacts directly with the target authentication
system. It can be detected, rate-limited, blocked, or cause account
lockouts.

### Q3. Explain offline password cracking.

Offline password cracking occurs when an attacker obtains password
hashes and tests candidate passwords locally without repeatedly
communicating with the original authentication system.

### Q4. Name any arguments in the Hashcat command used to crack the password.

Examples:

``` text
-m 100
-r pass.rule
```

`-m 100` specifies the SHA-1 hash mode, while `-r pass.rule` specifies
the rule file.

### Q5. Explain the `hashid` command.

`hashid` is used to identify the likely hash algorithm/type of a given
hash. In this experiment:

``` bash
hashid pass.txt
```

identifies the hash as SHA-1.

## 21. Important Commands -- Quick Reference

``` bash
mkdir -p ~/Desktop/Exp9
cd ~/Desktop/Exp9

echo "01F3273F68195C29A1A2365BE7AD2B1AAD469A73" > pass.txt

hashid pass.txt

cd /usr/share/wordlists/
sudo gzip -dk rockyou.txt.gz

cd ~/Desktop/Exp9

touch pass.rule

python3 -c 'digits="0123456789abcdef"; base="c sa4 sb6 se3 sg9 si1 so0 ss5 st7 sz2"; f=open("pass.rule","w"); [f.write(base+" $"+j+"$@ ^"+i+"\n") for i in digits for j in digits]; f.close()'

wc -l pass.rule

sudo hashcat -m 100 pass.txt /usr/share/wordlists/rockyou.txt -r pass.rule

hashcat -m 100 pass.txt --show
```

## 22. Final Directory Structure

``` text
~/Desktop/Exp9/
│
├── pass.txt
└── pass.rule
```

`pass.txt` contains the given hash, while `pass.rule` contains the 256
generated Hashcat rules.
