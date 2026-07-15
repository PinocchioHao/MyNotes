
### 使用过git的哪些高级功能 advanced Git features？
I have used **stash** to temporarily save my code changes when switching branches. I have used **rebase** to keep the branch history clean and linear when pulling updates, but improper use of rebase can **mess up** the branch. In my bank, we value **stability**, so we follow coding rules and always use **merge**. I also used **cherry-pick** once to **pick specific commits for an urgent release.**



### How do you use Git and CI/CD (Continuous Integration/ Continuous Deployment/ Delivery)?
We create a branch from the last **production version** on master, develop locally, and then pull into the **release branch**. After all changes are ready, we merge it to master branch.  
Then we use our **internal pipeline** to **build**, check **test coverage**, run **code quality checks**, and finally **deploy** to production. We also update the **config center** and run **DB scripts** if needed.


### ❓**Have you ever had problems using Git? How did you solve them?**

**✅ Answer:**

> Yes, our team had issues when several groups worked in parallel. Sometimes different versions caused conflicts or even overwriting.  
> To solve this, we always developed based on stable code. After each release, we merged unfinished work carefully.  
> Also, our team leader reviewed all merges, and we reminded each other to avoid mistakes.

