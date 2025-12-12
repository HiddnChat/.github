# Contributing to HiddnChat

Thank you for your interest in contributing to HiddnChat! This document provides guidelines for contributing to any of the HiddnChat repositories.

## Code of Conduct

By participating in this project, you agree to maintain a respectful and inclusive environment. We expect all contributors to:

- Be respectful and considerate
- Welcome newcomers and help them learn
- Focus on what is best for the community
- Accept constructive criticism gracefully

## Getting Started

### Prerequisites

See the [Developer Setup Guide](docs/DEVELOPMENT.md) for environment requirements.

### Finding Issues to Work On

1. Check the issue tracker in each repository
2. Look for issues labeled `good first issue` for beginners
3. Issues labeled `help wanted` are ready for contributors
4. Comment on an issue before starting work

## Development Workflow

### 1. Fork and Clone

```bash
# Fork the repository on GitHub, then:
git clone https://github.com/YOUR-USERNAME/api.git
cd api
git remote add upstream https://github.com/HiddnChat/api.git
```

### 2. Create a Branch

```bash
# Update main
git checkout main
git pull upstream main

# Create feature branch
git checkout -b feature/issue-42-add-presence-endpoint
```

**Branch naming conventions:**

| Type | Format | Example |
|------|--------|---------|
| Feature | `feature/issue-N-description` | `feature/issue-42-add-presence` |
| Bug fix | `fix/issue-N-description` | `fix/issue-99-auth-error` |
| Documentation | `docs/description` | `docs/update-readme` |
| Refactor | `refactor/description` | `refactor/simplify-auth` |

### 3. Make Changes

- Write clean, readable code
- Follow existing code style
- Add tests for new functionality
- Update documentation if needed

### 4. Commit Changes

We follow [Conventional Commits](https://www.conventionalcommits.org/):

```bash
git commit -m "feat: add user presence endpoint

- Add GET /users/{id}/presence
- Add POST /users/presence/bulk
- Include tests and documentation

Closes #42"
```

**Commit types:**

| Type | Description |
|------|-------------|
| `feat` | New feature |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `style` | Code style (formatting, semicolons) |
| `refactor` | Code change that neither fixes nor adds |
| `test` | Adding or updating tests |
| `chore` | Build process, dependencies |

### 5. Push and Create PR

```bash
git push origin feature/issue-42-add-presence-endpoint
```

Then create a Pull Request on GitHub.

## Pull Request Guidelines

### PR Title

Use the same format as commit messages:

```
feat: add user presence endpoint
```

### PR Description Template

```markdown
## Summary
Brief description of changes.

## Changes
- Added X
- Updated Y
- Fixed Z

## Testing
How to test these changes.

## Related Issues
Closes #42
```

### PR Checklist

Before submitting:

- [ ] Code follows project style guidelines
- [ ] Tests pass locally
- [ ] New code has test coverage
- [ ] Documentation updated if needed
- [ ] No merge conflicts with main
- [ ] Meaningful commit messages

## Code Style

### PHP (API)

- PSR-12 coding standard
- Run `vendor/bin/php-cs-fixer fix` before committing
- Use PHP 8.4 features appropriately
- Type hints required

```php
// Good
public function getUser(Uuid $id): ?User
{
    return $this->userRepository->find($id);
}

// Bad
function getUser($id)
{
    return $this->userRepository->find($id);
}
```

### TypeScript (MLS API, Web)

- ESLint + Prettier configuration
- Run `npm run lint` before committing
- Use strict TypeScript settings

```typescript
// Good
async function getUser(id: string): Promise<User | null> {
  return await userRepository.findById(id);
}

// Bad
async function getUser(id) {
  return await userRepository.findById(id);
}
```

### Rust (MLS Core)

- Follow Rust idioms
- Run `cargo fmt` before committing
- Run `cargo clippy` for lints

```rust
// Good
fn get_user(id: &str) -> Result<Option<User>, Error> {
    user_repository.find_by_id(id)
}
```

## Testing

### Writing Tests

- Test new features thoroughly
- Cover edge cases
- Use meaningful test names

```php
// PHP example
public function test_user_can_register_with_valid_credentials(): void
{
    // Arrange
    $data = ['username' => 'testuser', 'password' => 'SecurePass123'];

    // Act
    $response = $this->post('/api/auth/register', $data);

    // Assert
    $response->assertStatus(201);
}
```

### Running Tests

```bash
# PHP
php bin/phpunit

# TypeScript
npm test

# Rust
cargo test
```

## Documentation

### Code Documentation

- Document public APIs
- Explain complex logic
- Keep comments current

```php
/**
 * Get user presence status.
 *
 * @param Uuid $userId The user's unique identifier
 * @return Presence|null User's presence or null if not found
 * @throws AuthorizationException If not authorized
 */
public function getPresence(Uuid $userId): ?Presence
```

### README Updates

Update relevant README when:
- Adding new features
- Changing configuration
- Modifying setup steps

## Security

### Reporting Security Issues

**Do not** report security vulnerabilities through public issues.

Email: security@hiddn.chat

### Security Best Practices

- Never commit secrets
- Validate all input
- Use parameterized queries
- Follow least privilege principle

## Review Process

1. Automated CI checks must pass
2. At least one maintainer review required
3. Address all review comments
4. Maintainer merges when approved

### Response Times

- Initial response: 1-3 days
- Review completion: 1-7 days
- Complex PRs may take longer

## Community

### Getting Help

- Open a Discussion on GitHub
- Check existing issues and docs
- Be patient and respectful

### Recognition

Contributors are recognized in:
- Release notes
- Contributors list
- Commit co-author credits

## License

By contributing, you agree that your contributions will be licensed under the same license as the project (Proprietary).

---

Thank you for contributing to HiddnChat!
