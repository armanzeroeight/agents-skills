---
name: branch-strategy-advisor
description: Recommends git branching strategies and workflows based on team size, deployment frequency, and project requirements. Use when choosing branching models, planning workflows, or optimizing git processes.
---

# Branch Strategy Advisor

Recommend optimal branching strategies for your project.

## Quick Start

Choose strategy based on team and deployment:
- Small team + continuous deployment → GitHub Flow
- Large team + scheduled releases → Gitflow
- Fast iteration + feature flags → Trunk-Based Development

## Instructions

### Step 1: Assess Project Context

**Team Size**:
- Small (1-5 developers)
- Medium (6-20 developers)
- Large (20+ developers)

**Deployment Frequency**:
- Continuous (multiple times per day)
- Daily
- Weekly
- Scheduled releases (monthly/quarterly)

**Release Model**:
- Single production version
- Multiple versions in production
- Long-term support versions
- Rolling releases

**Technical Constraints**:
- Feature flags available?
- Automated testing coverage?
- CI/CD pipeline maturity?
- Code review requirements?

### Step 2: Select Branching Strategy

**GitHub Flow** - Best for:
- Small to medium teams
- Web applications
- Continuous deployment
- Simple workflow needed

**Structure**:
```
main (production)
  ├── feature/user-auth
  ├── feature/payment-integration
  └── bugfix/login-error
```

**Workflow**:
1. Create feature branch from main
2. Make changes and commit
3. Open pull request
4. Review and test
5. Merge to main
6. Deploy automatically

**Pros**: Simple, fast, continuous deployment
**Cons**: Requires good CI/CD, no release staging

---

**Gitflow** - Best for:
- Large teams
- Scheduled releases
- Multiple versions in production
- Need for hotfix support

**Structure**:
```
main (production)
develop (integration)
  ├── feature/new-dashboard
  ├── feature/api-v2
  ├── release/v2.0
  └── hotfix/critical-bug
```

**Workflow**:
1. Feature branches from develop
2. Merge features to develop
3. Create release branch from develop
4. Test and fix in release branch
5. Merge release to main and develop
6. Tag release on main
7. Hotfixes from main, merge to main and develop

**Pros**: Structured, supports multiple versions, clear release process
**Cons**: Complex, slower, more overhead

---

**Trunk-Based Development** - Best for:
- High-performing teams
- Continuous integration
- Feature flags available
- Fast iteration needed

**Structure**:
```
main (trunk)
  ├── short-lived-feature-1
  └── short-lived-feature-2
```

**Workflow**:
1. Create short-lived branch (< 1 day)
2. Make small changes
3. Merge to main frequently
4. Use feature flags for incomplete features
5. Deploy main continuously

**Pros**: Fast integration, simple, encourages small changes
**Cons**: Requires discipline, needs feature flags, high test coverage

---

**GitLab Flow** - Best for:
- Environment-based deployments
- Need for staging
- Hybrid approach
- Multiple environments

**Structure**:
```
main (development)
  ├── feature/new-feature
  ├── staging (staging environment)
  └── production (production environment)
```

**Workflow**:
1. Feature branches from main
2. Merge to main (development)
3. Merge main to staging for testing
4. Merge staging to production for release
5. Hotfixes can go directly to production

**Pros**: Environment-based, flexible, clear promotion path
**Cons**: More branches to manage, can be slower

### Step 3: Define Branch Naming

**Recommended conventions**:

```
feature/short-description
feature/TICKET-123-description
feature/user-auth-jwt

bugfix/short-description
bugfix/ISSUE-456-null-pointer
bugfix/login-validation

hotfix/critical-issue
hotfix/security-patch
hotfix/payment-failure

release/version
release/v2.0.0
release/2024-Q1

chore/maintenance-task
chore/update-dependencies
chore/cleanup-logs
```

**Naming rules**:
- Use lowercase with hyphens
- Include ticket/issue number if applicable
- Be descriptive but concise
- Use consistent prefixes
- Avoid special characters

### Step 4: Set Branch Policies

**Main/Master Branch**:
- Require pull request reviews (1-2 reviewers)
- Require status checks to pass
- Require up-to-date branches
- Restrict direct pushes
- Require signed commits (optional)

**Develop Branch** (if using Gitflow):
- Require pull request reviews
- Require CI to pass
- Allow direct pushes for hotfixes (optional)

**Feature Branches**:
- No restrictions
- Delete after merge
- Rebase on parent regularly

**Release Branches**:
- Restrict to release managers
- Only bug fixes allowed
- No new features

### Step 5: Define Merge Strategy

**Merge Commit** - When to use:
- Want to preserve branch history
- Multiple commits represent logical units
- Need audit trail
- Gitflow workflow

**Squash and Merge** - When to use:
- Want clean main branch history
- Many WIP commits in feature branch
- Single logical change
- GitHub Flow workflow

**Rebase and Merge** - When to use:
- Want linear history
- No merge commits
- Team comfortable with rebasing
- Trunk-Based Development

**Fast-Forward Only** - When to use:
- Strict linear history
- Small, atomic changes
- Trunk-Based Development
- Advanced teams

## Strategy Comparison

| Strategy | Team Size | Deployment | Complexity | Best For |
|----------|-----------|------------|------------|----------|
| GitHub Flow | Small-Medium | Continuous | Low | Web apps, SaaS |
| Gitflow | Medium-Large | Scheduled | High | Enterprise, versioned products |
| Trunk-Based | Any | Continuous | Medium | High-performing teams |
| GitLab Flow | Medium | Regular | Medium | Multi-environment deployments |

## Common Scenarios

**Scenario: Startup with 3 developers**
- **Recommendation**: GitHub Flow
- **Rationale**: Simple, fast, supports continuous deployment
- **Setup**: main branch + feature branches + PR reviews

**Scenario: Enterprise with 50 developers**
- **Recommendation**: Gitflow
- **Rationale**: Structured, supports multiple releases, clear process
- **Setup**: main + develop + feature/release/hotfix branches

**Scenario: SaaS with daily deployments**
- **Recommendation**: Trunk-Based Development
- **Rationale**: Fast integration, continuous deployment, small changes
- **Setup**: main branch + short-lived features + feature flags

**Scenario: Mobile app with app store releases**
- **Recommendation**: Gitflow or GitLab Flow
- **Rationale**: Scheduled releases, need for release branches, testing period
- **Setup**: Release branches for app store submissions

**Scenario: Open source project**
- **Recommendation**: GitHub Flow
- **Rationale**: Simple for contributors, PR-based, clear process
- **Setup**: main branch + fork-based contributions

## Migration Strategies

**From no strategy to GitHub Flow**:
1. Protect main branch
2. Require PR reviews
3. Set up CI/CD
4. Document workflow
5. Train team

**From GitHub Flow to Gitflow**:
1. Create develop branch from main
2. Update CI to test develop
3. Create first release branch
4. Document new workflow
5. Migrate features to new model

**From Gitflow to Trunk-Based**:
1. Merge develop to main
2. Remove develop branch
3. Implement feature flags
4. Shorten branch lifetimes
5. Increase deployment frequency

## Best Practices

**Branch Hygiene**:
- Delete merged branches automatically
- Limit active branches per developer (2-3)
- Keep branches short-lived (< 3 days)
- Sync with parent branch daily
- Use descriptive names

**Code Review**:
- Review within 24 hours
- Keep PRs small (< 400 lines)
- Provide constructive feedback
- Test before approving
- Use review checklists

**CI/CD Integration**:
- Run tests on every push
- Deploy main automatically (if continuous)
- Use staging environments
- Implement rollback mechanisms
- Monitor deployments

**Documentation**:
- Document chosen strategy in README
- Create workflow diagrams
- Provide examples
- Update as strategy evolves
- Include troubleshooting guide

## Advanced

For complex scenarios:
- Monorepo branching strategies
- Multi-team coordination
- Release train models
- Hotfix procedures
- Branch protection rules
