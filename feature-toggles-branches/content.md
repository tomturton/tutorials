If you manage a medium to large-sized project then the chances are you will be regularly updating it: adding new features or refactoring the code.
But while you are developing a new feature, there's always the possibility that you have to urgently fix a bug in the production version.
The problem is, you are half-way through some major work. You can't quickly make the fix on top of these changes and hope everything works. That would be a recipe for disaster.

So how do you manage adding new features if there's always a risk you need to quickly push fixes?


## Version control systems

This problem is solved by managing your code in a version control system, such as [Git](https://git-scm.com/) or [Subversion](https://subversion.apache.org/).
If you aren't familiar with this idea, then think of it as a way of keeping a history of the project, while having several 'branches' representing other stages of the release cycle, which you can switch between at any time.

In this scenario you simply switch to your master branch, clone it and fix the problem, then merge it back into the master again and tag with a new version number. Your development code is still seperate from the live version (but don't forget to merge your fix into it).

But sometimes you might want to work on several features at once; some big and some small. You may wish to release these features as and when they are ready. What is the best approach to take here?


## Feature branches

Typically, in these sort of projects, you would have several branches in your VCS.

If we're using Vincent Driessen's [Git flow model](http://nvie.com/posts/a-successful-git-branching-model/), there would be branches for releases, hotfixes, and general development.
For major feature development, or work on future releases, there would be additional branches for each new feature.

I love this way of working, as it keeps a project well organised. Each branch has an explicit purpose and the release cycle is very clear.


### Pros

- Multiple features can be worked on simultaneously, and can be merged back into the main development branch as and when they are ready.

### Cons

- Merging can become a **nightmare**. The more that two branches are developed in parallel, the harder it will be to merge them.
- You (and other developers) would need to be fairly proficient at using your VCS, and would all need a clear understanding of the branching model you are using.



## Feature toggles

Perhaps a more controversial approach would be to have a single branch for project development, but build in a toggle system for each new feature.

While you are developing, you will want your experimental code to be active; but not if in the meanwhile you decide to push an update to production.

This would work by having a configuration file or database that defines each feature and specifies if it is on or off.
You would then wrap the new features in conditionals to determine if the feature should be active or not.

### Pros

- Simple to understand and use
- Feature toggling is very similar to ACL, so if you already have a system in place to give different users access to particular features, then it may be very quick to implement.

### Cons

- When a feature is finished, tested and ready for production, you should take the time to remove all the conditionals that allowed you to toggle the feature, in order to decrease clutter and prevent confusion later.
- Special care will have to be taken to not break existing code while developing a feature, as you don't want to be releasing anything unfinished or untested that the production code relies upon. Consequently, feature toggling should be limited to new features. Code refactoring should still be done in seperate branches.
- You will have to be careful not to accidently expose unfinished features in the production version.


### Implementing feature toggle

Whatever language you are working with, there will likely be tried and tested libraries to incorporate feature toggling, which often include UIs.

You may prefer to write something yourself.
It doesn't have to be complicated, but I would definitely consider having different configurations for each environment.
Each developer will want control of which features to toggle and you don't want to accidently enable an unfinished feature in production.
This may rule out using an existing ACL component however.


## Conclusion

I'm not sure there is definitive approach for every project or developer.

Certainly try both methods, and use your experiences to help you choose on a project-by-project basis.

I would speculate that feature toggling may be nice for small and medium-sized projects, but for anything large that is in constant development, it may be safer to use feature branching.



*[VCS]: Version control system
*[ACL]: Access control list
