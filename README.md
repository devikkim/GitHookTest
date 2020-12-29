# GitHookTest
This repository is test about git hook to ```prepare-commit-msg``` ([more information](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks))

If you applied this githook, then you can get a prefix(you specified) on your commit message.


## Where is?
There is prepare-commit-msg file in `../.git/hooks/` with other git hook files.

```
# your_repository/.git/hooks/ls

applypatch-msg.sample     pre-applypatch.sample     pre-rebase.sample
commit-msg.sample         pre-commit.sample         pre-receive.sample
fsmonitor-watchman.sample pre-merge-commit.sample   prepare-commit-msg.sample
post-update.sample        pre-push.sample           update.sample
```

## How to?
Remove `.sample` keyword at `prepare-commit-msg.sample` and then let's enter the following command to gain executable privileges.

```
chmod +x .git/hooks/prepare-commit-msg
```

And then open the file using vi editor or text editor, etc

You can see following

```
#!/bin/sh
#
# An example hook script to prepare the commit log message.
# Called by "git commit" with the name of the file that has the
# commit message, followed by the description of the commit
# message's source.  The hook's purpose is to edit the commit
# message file.  If the hook fails with a non-zero status,
# the commit is aborted.
#
# To enable this hook, rename this file to "prepare-commit-msg".

# This hook includes three examples. The first one removes the
# "# Please enter the commit message..." help message.
#
# The second includes the output of "git diff --name-status -r"
# into the message, just before the "git status" output.  It is
# commented because it doesn't cope with --amend or with squashed
# commits.
#
# The third example adds a Signed-off-by line to the message, that can
# still be edited.  This is rarely a good idea.

COMMIT_MSG_FILE=$1
COMMIT_SOURCE=$2
SHA1=$3

/usr/bin/perl -i.bak -ne 'print unless(m/^. Please enter the commit message/..m/^#$/)' "$COMMIT_MSG_FILE"
```

Add the following below the code above.

```
# This way you can customize which branches should be skipped when
# prepending commit message.
if [ -z "$BRANCHES_TO_SKIP" ]; then
  BRANCHES_TO_SKIP=(master develop)
fi

# BRANCH_NAME will be "jira/TEST-000001"
BRANCH_NAME=$(git symbolic-ref --short HEAD)

# BRANCH_TYPE will be "jira"
BRANCH_TYPE="${BRANCH_NAME%%/*}"

# Specified the branch to get prefix.
if [ ${BRANCH_TYPE} != "jira" ]; then
  exit 0
fi

# BRANCH_NAME will be "TEST-000001"
BRANCH_NAME="${BRANCH_NAME##*/}"

# JIRA_ID will be "TEST-000001" it's will be prefix tag such as [TEST-000001]
JIRA_ID=`echo $BRANCH_NAME | egrep -o 'TEST-[0-9]+'`

BRANCH_EXCLUDED=$(printf "%s\n" "${BRANCHES_TO_SKIP[@]}" | grep -c "^$BRANCH_NAME$")
BRANCH_IN_COMMIT=$(grep -c "$JIRA_ID" $1)

if [ -z $JIRA_ID ]; then
  exit 0
fi

if [ -n $JIRA_ID ] && ! [[ $BRANCH_EXCLUDED -eq 1 ]] && ! [[ $BRANCH_IN_COMMIT -ge 1 ]]; then
  sed -i.bak -e "1s/^/[${JIRA_ID}] /" $1
fi
```

## Test
Now you can see the prefix when after you tried to commit message. 

```
>> echo "test commit" > commit1.txt
>> git add commit1.txt
>> git commit -m "for check to prefix"

>> git log --oneline

66f4f72 (HEAD -> jira/TEST-000001) [TEST-000001] for check to prefix
```
