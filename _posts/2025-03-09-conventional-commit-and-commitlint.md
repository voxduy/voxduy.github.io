---
title: Conventional commits and commitlint - A Guide to Cleaner Git Histories
author: voxduy
date: 2025-03-09 00:45:00 +0700
categories: [Sharing]
tags: [Git]
image:
  path: /posts/2025-03-09-conventional-commit-and-commitlint/git_version_control.jpeg
  width: 800
  height: 500
pin: false
---

## Introduction

When working on a software project, maintaining a clean and structured Git history is crucial for collaboration, debugging, and automation. One of the best ways to achieve this is by adopting a consistent commit message format. This is where **Conventional Commits** and **commitlint** come in.

In this post, we’ll explore what Conventional Commits are, why they matter, and how commitlint can help enforce them.

## What Are Conventional Commits?

Conventional Commits is a specification that provides a standard format for writing commit messages. It helps make Git history more readable and allows for automation in tools like changelogs, versioning, and CI/CD workflows.

A Conventional Commit message follows this structure:

```bash
<type>[optional scope]: <description>

[optional body]

[optional footer]
```

### Example Commit Messages

```bash
feat(auth): add OAuth2 support
fix(ui): resolve button alignment issue
chore(deps): update dependencies
docs: add new document
```

### Common Types in Conventional Commits

- **feat**: A new feature
- **fix**: A bug fix
- **docs**: Documentation updates
- **style**: Code style changes (whitespace, formatting, etc.)
- **refactor**: Code changes that neither fix a bug nor add a feature
- **test**: Adding or modifying tests
- **chore**: Maintenance tasks (dependency updates, build changes)

### Why Use Conventional Commits?

- **Improved Readability**: Easier to understand commit history
- **Automated Versioning**: Works well with tools like Semantic Versioning (SemVer)
- **Better Collaboration**: Helps developers and teams quickly grasp changes
- **Enhanced CI/CD**: Enables automation in deployments, changelogs, and release notes

## Enforcing Conventional Commits with commitlint

To ensure that commit messages follow the Conventional Commits format, we can use **commitlint**. It is a tool that lints commit messages and prevents commits that don’t follow the convention.

### Setting Up commitlint

1. **Install commitlint and husky** (to enforce rules in Git hooks):
   ```sh
   npm install --save-dev @commitlint/{config-conventional,cli} husky
   ```

2. **Create a configuration file (`commitlint.config.js`)**:
   ```js
   module.exports = {
     extends: ['@commitlint/config-conventional']
   };
   ```

3. **Set up husky to run commitlint on commits**:
   ```sh
   npx husky install
   npx husky add .husky/commit-msg 'npx --no-install commitlint --edit "$1"'
   ```

4. **Verify the setup**:
   ```sh
   echo "feat: add new API endpoint" | npx commitlint
   ```
   If the message is valid, there will be no error. Otherwise, commitlint will display an error message.

### Additional Customization

commitlint allows you to customize rules by modifying `commitlint.config.js`. For example, to enforce a maximum subject length of 72 characters:

```js
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'header-max-length': [2, 'always', 72]
  }
};
```

## Conclusion

Using Conventional Commits and commitlint ensures a cleaner, more maintainable Git history. By enforcing structured commit messages, teams can improve collaboration, automate versioning, and integrate seamlessly with CI/CD pipelines.

Adopting these best practices is a small investment that pays off in the long run, leading to better-organized projects and efficient software development workflows.
