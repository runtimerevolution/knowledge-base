# Chapter I - Continuous Delivery
`(avr. time for this chapter: 2 days)`

Now that your project is automatically validated through Continuous Integration, the next step is to automate the deployment process. This practice is known as Continuous Delivery (CD).

In this chapter, you will extend your CI pipeline to include automated deployments.

## Platform Selection

Use the same platform you configured for Continuous Integration:

- [GitHub Actions](https://docs.github.com/en/actions)
- [Bitbucket Pipelines](https://support.atlassian.com/bitbucket-cloud/docs/get-started-with-bitbucket-pipelines/)
- [GitLab CI/CD](https://docs.gitlab.com/ee/ci/)

## Deployment Strategy

Before implementing automated deployments, consider the following important principles:

### Key Considerations

1. **Selective Deployment**: Not every commit should trigger a deployment
2. **Avoid Redundancy**: Do not repeat validation steps that were already executed in CI
3. **Branch Strategy**: Define which branches trigger deployments

### Git Branching Strategies

Choose a branching strategy that fits your workflow. Common approaches include:

- **Git Flow**: Separate branches for features, releases, and hotfixes
- **GitHub Flow**: Simple workflow with feature branches and main
- **Trunk-Based Development**: Short-lived branches merged frequently to main

> Note: Discuss branching strategies with your tutor if you need guidance.

## Pipeline Configuration

### Steps to implement:

1. Configure deployment to trigger only on specific branches (e.g., `main`, `production`)
2. Ensure CI validation passes before deployment begins
3. Set up deployment credentials securely using environment variables or secrets
4. Configure deployment to your hosting platform (Render, Heroku, etc.)
5. (Optional) Implement staging and production environments with separate deployment rules

### Deployment Targets

If you deployed your application in previous chapters, configure automated deployment to the same platform:

- [Render](https://render.com)
- [Heroku](https://heroku.com)
- [Railway](https://railway.app)
- Other platform of your choice

## Best Practices

- Store sensitive credentials as encrypted secrets, never in code
- Implement rollback procedures for failed deployments
- Consider implementing deployment notifications (Slack, email, etc.)
- Test your deployment pipeline with non-critical changes first
