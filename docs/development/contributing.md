# Contributing Guide

Thank you for your interest in contributing to the Leichte Sprache API! This guide will help you get started with development and understand our contribution process.

## Quick Start for Contributors

1. **Fork and clone** the repository
2. **Set up** your development environment
3. **Make changes** following our guidelines
4. **Test** your changes thoroughly
5. **Submit** a pull request

## Development Setup

### Prerequisites

- **Python 3.11+** (Python 3.12 recommended)
- **Git** for version control
- **uv** (recommended) or pip for package management
- **8GB+ RAM** for running ML models

### Environment Setup

```bash
# 1. Fork the repository on GitHub
# 2. Clone your fork
git clone https://github.com/YOUR_USERNAME/leichte-sprache.git
cd leichte-sprache

# 3. Set up development environment
uv venv
source .venv/bin/activate
uv sync --extra dev

# 4. Install pre-commit hooks
pre-commit install

# 5. Download required models
uv run python -m spacy download de_core_news_lg

# 6. Run tests to verify setup
python test-suite/test_runner.py
```

### Development Dependencies

Additional tools for development:

```bash
# Code formatting and linting
ruff check .
black --check .

# Type checking
mypy .

# Documentation
mkdocs serve

# Testing
pytest
```

## Project Structure

Understanding the codebase layout:

```
leichte-sprache/
├── api_main.py              # FastAPI application
├── analysis_service.py      # Core analysis service
├── config.py               # Application configuration
│
├── regeln/                 # Rule modules (18 rules)
│   ├── fremdwoerter/       # Foreign words rule
│   ├── satzlaenge/         # Sentence length rule
│   └── ...                 # Other rules
│
├── tools/                  # Utilities and CLI tools
│   ├── agent_optimizer.py  # LLM generation engine
│   └── ml/                 # ML training tools
│
├── test-suite/             # Test framework
│   ├── test_runner.py      # Test runner
│   └── [rule_name]/        # Test cases per rule
│
├── docs/                   # Documentation (MkDocs)
├── prompts/               # LLM prompts for generation
└── scripts/               # Deployment and utility scripts
```

## Types of Contributions

We welcome various types of contributions:

### 🐛 Bug Reports

Found a bug? Help us fix it:

1. **Check existing issues** to avoid duplicates
2. **Use the bug report template**
3. **Provide minimal reproduction steps**
4. **Include system information**

### ✨ Feature Requests

Have an idea for improvement?

1. **Check if it's already requested**
2. **Explain the use case clearly**
3. **Consider backward compatibility**
4. **Discuss implementation approach**

### 📝 Documentation

Documentation improvements are always welcome:

- Fix typos and grammatical errors
- Improve clarity and examples
- Add missing documentation
- Translate content (German/English)

### 🔧 Code Contributions

#### Adding New Rules

The most common contribution type. See the Rule Development section below for details.

#### Core Improvements

- API enhancements
- Performance optimizations
- Testing improvements
- CI/CD pipeline updates

## Development Guidelines

### Code Style

We use automated formatting and linting:

```bash
# Format code
black .
ruff --fix .

# Check style
ruff check .
black --check .
mypy .
```

#### Python Style Guide

- **PEP 8** compliance (enforced by Black)
- **Type hints** for all public functions
- **Docstrings** for classes and functions
- **Clear variable names** in German or English

#### Example Function

```python
def pruefe_regel(doc: spacy.tokens.Doc) -> List[str]:
    """
    Prüft Text auf Verstöße gegen die Satzlänge-Regel.

    Args:
        doc: SpaCy-verarbeitetes Dokument

    Returns:
        Liste von Verstößen als String-Beschreibungen

    Raises:
        ValueError: Wenn das Dokument ungültig ist
    """
    violations = []

    for sent in doc.sents:
        word_count = len([token for token in sent if not token.is_punct])
        if word_count > 15:
            violations.append(f"Satz zu lang: {word_count} Wörter (max. 15)")

    return violations
```

### Testing Requirements

All contributions must include tests:

#### Rule Tests

Each rule must have comprehensive test cases:

```bash
# Test specific rule
python test-suite/test_runner.py satzlaenge

# Add test case in test-suite/satzlaenge/
# Format: test_name.txt
# expected: flag|pass
# description: Test description

Test text goes here.
```

#### API Tests

For API changes:

```python
# tests/test_api.py
def test_analyse_endpoint():
    response = client.post("/analyse", json={"text": "Test text."})
    assert response.status_code == 200
    assert "annotated_text" in response.json()
```

#### Performance Tests

For performance-critical changes:

```bash
# Benchmark before and after
python -m tools.benchmark --rule satzlaenge --iterations 100
```

### Git Workflow

We use the GitHub Flow:

```bash
# 1. Create feature branch
git checkout -b feature/add-punctuation-rule

# 2. Make changes and commit
git add .
git commit -m "feat: add punctuation rule for Leichte Sprache compliance"

# 3. Push to your fork
git push origin feature/add-punctuation-rule

# 4. Open pull request on GitHub
```

#### Commit Message Format

We follow [Conventional Commits](https://www.conventionalcommits.org/):

```
type(scope): description

[optional body]

[optional footer]
```

**Types:**
- `feat`: New features
- `fix`: Bug fixes
- `docs`: Documentation changes
- `style`: Code style changes
- `refactor`: Code refactoring
- `test`: Test additions/changes
- `chore`: Maintenance tasks

**Examples:**
```
feat(rules): add punctuation validation rule
fix(api): handle empty text input correctly
docs: update installation guide for uv
test: add test cases for compound word detection
```

## Pull Request Process

### Before Submitting

1. **Sync with main branch**:
   ```bash
   git checkout main
   git pull upstream main
   git checkout feature/your-branch
   git rebase main
   ```

2. **Run full test suite**:
   ```bash
   python test-suite/test_runner.py
   ruff check .
   black --check .
   mypy .
   ```

3. **Update documentation** if needed

4. **Add changelog entry** if significant

### PR Template

Use this template for your pull request:

```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Testing
- [ ] Tests pass locally
- [ ] New tests added
- [ ] Manual testing completed

## Checklist
- [ ] Code follows style guidelines
- [ ] Self-review completed
- [ ] Documentation updated
- [ ] No breaking changes (or documented)
```

### Review Process

1. **Automated checks** must pass (CI/CD)
2. **Code review** by maintainers
3. **Testing** in development environment
4. **Merge** when approved

## Rule Development

### Rule Structure

Each rule follows this structure:

```
regeln/rule_name/
├── __init__.py          # Exports pruefe_regel
├── regel.py             # Main rule implementation
├── config.py           # Configuration settings
├── README.md           # Rule documentation
└── model/              # ML models (if needed)
    ├── model.pkl
    └── tokenizer.pkl
```

### Rule Implementation

```python
# regeln/example_rule/regel.py
import spacy
from typing import List

def pruefe_regel(doc: spacy.tokens.Doc) -> List[str]:
    """
    Implement your rule logic here.

    Args:
        doc: spaCy document object

    Returns:
        List of violation messages
    """
    violations = []

    # Your rule logic here
    for token in doc:
        if your_condition(token):
            violations.append(f"Violation: {token.text}")

    return violations

def your_condition(token) -> bool:
    """Helper function for rule logic."""
    return token.pos_ == "NOUN" and len(token.text) > 20
```

### Rule Configuration

```python
# regeln/example_rule/config.py
REGEL_NAME = "example_rule"
REGEL_BESCHREIBUNG = "Checks for example violations"
REGEL_KATEGORIE = "lexical"  # syntax, lexical, stylistic, technical
REGEL_AKTIVIERT = True
REGEL_PRIORITAET = 5  # 1-10, higher = more important
```

### Rule Documentation

```markdown
# Example Rule

## Purpose
Brief description of what this rule checks.

## Leichte Sprache Guidelines
Reference to official guidelines this rule implements.

## Examples

### Violations
- Input: "Problematic text"
- Output: "Violation message"

### Correct Usage
- Input: "Correct text"
- Output: No violations

## Configuration
List of configuration options.

## Implementation Notes
Technical details for developers.
```

## ML Model Contributions

### Training New Models

1. **Prepare training data** in the required format
2. **Use the ML training CLI**:
   ```bash
   python -m tools.ml train your_rule --data-path data/your_rule/
   ```
3. **Evaluate model performance**:
   ```bash
   python -m tools.ml evaluate your_rule --verbose
   ```
4. **Submit model with training logs**

### Model Requirements

- **Performance**: Minimum 80% accuracy on test set
- **Size**: Models should be < 100MB when possible
- **Format**: Use standardized formats (pickle, ONNX, HuggingFace)
- **Documentation**: Include training procedure and metrics

## Documentation Contributions

### MkDocs Setup

```bash
# Install documentation dependencies
pip install mkdocs-material mkdocs-awesome-pages-plugin

# Serve locally
mkdocs serve

# Build static site
mkdocs build
```

### Documentation Guidelines

1. **Clear structure**: Use proper headings and sections
2. **Code examples**: Include working examples
3. **Screenshots**: Add when helpful (store in docs/assets/)
4. **Links**: Use relative links for internal docs
5. **Language**: English for technical docs, German for user-facing content

### Writing Style

- **Concise and clear** explanations
- **Active voice** when possible
- **Step-by-step** instructions for procedures
- **Examples** for complex concepts

## Community Guidelines

### Code of Conduct

- **Be respectful** and inclusive
- **Help newcomers** learn and contribute
- **Give constructive feedback** in reviews
- **Acknowledge contributions** from others

### Communication

- **GitHub Issues**: Bug reports and feature requests
- **GitHub Discussions**: Questions and general discussion
- **Pull Request Comments**: Code-specific discussion

## Release Process

Maintainers handle releases, but contributors should know the process:

1. **Version bumping** follows semantic versioning
2. **Changelog** is updated with all changes
3. **Testing** in staging environment
4. **Docker images** are built and published
5. **Documentation** is deployed

## Getting Help

Stuck? Here's how to get help:

1. **Check documentation** first
2. **Search existing issues** for similar problems
3. **Ask in GitHub Discussions** for questions
4. **File an issue** for bugs or feature requests

### Common Questions

**Q: How do I add a new rule?**
A: See the Rule Development section in this guide for step-by-step instructions.

**Q: My tests are failing, what should I check?**
A: Ensure you have all models downloaded and your environment is properly set up.

**Q: How do I test API changes?**
A: Use the test suite and manual testing with curl or Swagger UI.

**Q: Where do I add my rule's test cases?**
A: Create a directory `test-suite/your_rule_name/` with `.txt` files for each test case.

## Recognition

Contributors are recognized in:

- **CONTRIBUTORS.md** file
- **Release notes** for significant contributions
- **Documentation** credits where appropriate

Thank you for contributing to making German text more accessible! 🎉