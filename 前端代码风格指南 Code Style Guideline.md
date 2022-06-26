#### [Git Style Guide](https://github.com/agis/git-style-guide)

##### Above all:

- Meaningful branch name and commit message
- Use `rebase` Instead of `merge` to keep commit history clean and simple：
    - When you develop alone: `git pull master` and `rebase` it onto your dev branch **before merge**.
    - When you develop with others: merge each dev branch to a common branch, and then 
deploy the common branch to environment you need. Rebase the common branch before merging each dev branch；Also remember to rebase master to common branch before release.

[Git push rejected after feature branch rebase](https://stackoverflow.com/questions/8939977/git-push-rejected-after-feature-branch-rebase)

#### JavaScript Style Guide

- [React Style Guide](https://github.com/airbnb/javascript/tree/master/react)
- [JavaScript Standard Style](https://github.com/standard/standard)：brief | plugins and config
- [JavaScript Clean Code](https://github.com/ryanmcdermott/clean-code-javascript)

Learn more Style Guides at [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript) :)

You could also sweep your eyes over [Google JavaScript Style Guide](https://google.github.io/styleguide/jsguide.html).


#### CSS Style Guide

- [Css-in-JavaScript](https://github.com/airbnb/javascript/tree/master/css-in-javascript)
- [CSS and Sass](https://github.com/airbnb/css)
