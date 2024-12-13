How can I merge content from master branch to develop branch

To merge content from the `master` branch into the `develop` branch in Git, you can follow these steps:

### Step-by-Step Instructions

1. **Ensure a Clean Working Directory**  
    Before merging, ensure there are no uncommitted changes in your working directory. Commit or stash any pending changes.
    
    ```bash
    git status
    ```
    
2. **Switch to the `develop` Branch**  
    You must be on the branch where you want to apply the merge (in this case, `develop`).
    
    ```bash
    git checkout develop
    ```
    
3. **Pull the Latest Changes**  
    To ensure you're working with the latest changes, pull the latest updates for both `develop` and `master`.
    
    ```bash
    git pull origin develop
    git pull origin master
    ```
    
4. **Merge `master` into `develop`**  
    Now, merge the `master` branch into the `develop` branch.
    
    ```bash
    git merge master
    ```
    
5. **Resolve Conflicts (if any)**  
    If there are conflicts during the merge, Git will notify you. Open the files with conflicts, resolve them, and mark them as resolved.
    
    ```bash
    git status  # Shows the files with conflicts
    git add <file>  # After resolving each conflict, stage the file
    ```
    
    Once all conflicts are resolved, complete the merge:
    
    ```bash
    git commit  # Finalizes the merge
    ```
    
6. **Push the Changes**  
    Push the updated `develop` branch back to the remote repository.
    
    ```bash
    git push origin develop
    ```
    

### Verification

- Check the history to ensure the merge was successful:
    
    ```bash
    git log --oneline --graph
    ```
    

This process merges the `master` branch changes into `develop` while maintaining both branch histories.