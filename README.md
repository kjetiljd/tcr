
# test && commit || revert (TCR)

TCR scripts - the way I do it.

This is written for a Gradle Wrapper project, and used on MacOS.

`tcr`: This is the gold.
```
buildIt && (testIt && success && commitIt || (failure && revertIt))
```

`buildIt`: Compile production and test code
```
./gradlew testClasses
```

`testIt`: Stage files and try to build with tests. Unstage if it fails.
```
git add -A && ./gradlew build || git reset HEAD -- .
```

`commitIt`: Open the commit dialog. I use Arlo's Commit Notation.
```
git commit
```

`revertIt`: I only revert the production code changes. I also put them in a stash - just in case.
```
git stash drop 0 2&>/dev/null; git add -A -- ':!src/main/' && git stash push --keep-index && git restore --staged -- ':!src/main/'
```

`success`: Notification when success - nice when using `buddy` below (uses a MacOS feature).
```
osascript -e 'display notification "Success" with title "Completed" subtitle "win!" sound name "Ping"'
```

`failure`: Notification when failure - nice when using `buddy` below (uses a MacOS feature).
```
osascript -e 'display notification "Failure" with title "Incomplete" subtitle "wtf!" sound name "Sosumi"'
exit 1
```

`buddy`: Run TCR on save. A really strict buddy that keeps you on the straight and narrow. (`brew install fswatch` needed)
```
while(true);
  do
    fswatch -1 src;
    tcr;
  done;
```

`collab`: Continuous integration.
```
while(true);
  do
    git pull --rebase;
    git push;
    sleep 30;
  done;
```
