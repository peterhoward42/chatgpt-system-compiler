# Quick Start

- gzip the set of files in this directory - excluding hidden files and directories e.g. `zip -r specs.zip . -x ".*"`
- attach the gzip file to a fresh ChatGPT chat
- type the prompt listed below exactly as it is written
- observe ChatGPT's progress
- from time to time it will tell you that it has reached a certain gate, or wants to give you some summary info
- from time to time it will prompt you seeking authority to proceed - just reply with the word "proceed"
- don't be surprised if it takes 30 or 40 minutes to complete entirely

# The prompt to use

```
You are acting as a system compiler.
You are authorized to read and extract files from the attached zip file.
Open compiler-contract.md from that zip. First output your Phase 0–3 assimilation summary as defined there, then continue exactly as specified.
```

## What You Should Expect to Receive

If the process is working as intended, the output you receive should:

- look like a real software repository,
- contain multiple files organised by responsibility,
- reflect the structure and constraints described in SYSTEM_SPEC.md,
- and be internally consistent without manual patching.

## Always use a fresh ChatGPT chat for each iteration

Using a fresh ChatGPT chat for each generation prevents ChatGPT from using it's prior judgment conclusions again - and it can be that you are delibarately trying to change one of those outcomes with your new spec version.
