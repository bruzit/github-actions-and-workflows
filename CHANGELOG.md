# Changelog

## [0.11.2](https://github.com/bruzit/github-actions-and-workflows/compare/v0.11.1...v0.11.2) (2026-10-08)

### Bug Fixes

* run gitleaks when megalinter fails ([0a058e7](https://github.com/bruzit/github-actions-and-workflows/commit/0a058e72b6ef01c70d81eefa445dab16d512a34a))

## [0.11.1](https://github.com/bruzit/github-actions-and-workflows/compare/v0.11.0...v0.11.1) (2026-10-07)

### Bug Fixes

* warn when a diverged major tag is skipped ([315c03a](https://github.com/bruzit/github-actions-and-workflows/commit/315c03a81a138968079f4f13c23e43feedfeb051))

## [0.11.0](https://github.com/bruzit/github-actions-and-workflows/compare/v0.10.0...v0.11.0) (2026-10-06)

### Features

* add gitleaks secret scan ([c1b78e9](https://github.com/bruzit/github-actions-and-workflows/commit/c1b78e94aa4be3a2d665025b6e2dc6d70795cfad))

## [0.10.0](https://github.com/bruzit/github-actions-and-workflows/compare/v0.9.2...v0.10.0) (2026-10-03)

### ⚠ BREAKING CHANGES

* remove the reusable semantic release workflow

### Features

* remove the reusable semantic release workflow ([d8a175c](https://github.com/bruzit/github-actions-and-workflows/commit/d8a175c418b7a3f29358e72ceccab8abc5b644ad))

## [0.9.2](https://github.com/bruzit/github-actions-and-workflows/compare/v0.9.1...v0.9.2) (2026-10-03)

### Bug Fixes

* never move a major tag backwards ([eb3a628](https://github.com/bruzit/github-actions-and-workflows/commit/eb3a628aeb29f4f5f099bc9580dee09ebe70613c))

## [0.9.1](https://github.com/bruzit/github-actions-and-workflows/compare/v0.9.0...v0.9.1) (2026-10-02)

### Bug Fixes

* split semantic release plugins on whitespace ([4ea955a](https://github.com/bruzit/github-actions-and-workflows/commit/4ea955af5f973fe37112589b51c08d00aa82ebc6))

## [0.9.0](https://github.com/bruzit/github-actions-and-workflows/compare/v0.8.1...v0.9.0) (2026-10-01)

### Features

* add semantic release composite action ([17ecf1f](https://github.com/bruzit/github-actions-and-workflows/commit/17ecf1ff92897fa4b6c4ad145fc35b1131188be4))

## [0.8.1](https://github.com/bruzit/github-actions-and-workflows/compare/v0.8.0...v0.8.1) (2026-09-27)

### Bug Fixes

* stop installing semantic-release-major-tag ([3f6b299](https://github.com/bruzit/github-actions-and-workflows/commit/3f6b299a51d51dd92e5ea761cc367c1b649b3eb4))

## [0.8.0](https://github.com/bruzit/github-actions-and-workflows/compare/v0.7.0...v0.8.0) (2026-09-26)

### Features

* move major tags in the semantic release workflow ([8b463cf](https://github.com/bruzit/github-actions-and-workflows/commit/8b463cfef7357c4455347812b4fa59f575125a82))

## [0.7.0](https://github.com/bruzit/github-actions-and-workflows/compare/v0.6.0...v0.7.0) (2026-09-26)

### Features

* expose megalinter as reusable workflow ([fa8798c](https://github.com/bruzit/github-actions-and-workflows/commit/fa8798caa6663a8aaa1123563a4df4bb22a3a5b3))

## [0.6.0](https://github.com/bruzit/github-actions-and-workflows/compare/v0.5.5...v0.6.0) (2026-09-16)

### Features

* enable yaml linting in megalinter ([8506bcf](https://github.com/bruzit/github-actions-and-workflows/commit/8506bcf9592ff344e6272e1e7f3cd87118e0177e))

### Bug Fixes

* include yamllint config in megalinter triggers ([c1cd681](https://github.com/bruzit/github-actions-and-workflows/commit/c1cd6816348b8bc658647385842b71415a068161))

## [0.5.5](https://github.com/bruzit/github-actions-and-workflows/compare/v0.5.4...v0.5.5) (2026-09-15)

### Bug Fixes

* **megalinter:** remove persist-credentials false to fix auto-commit ([bf8dfda](https://github.com/bruzit/github-actions-and-workflows/commit/bf8dfdae90a2151655ef14fb0d9b8e2177aba694))
* **megalinter:** suppress zizmor artipacked on checkout, grant pull-requests write ([03d42f2](https://github.com/bruzit/github-actions-and-workflows/commit/03d42f290feaba99d1aebcb7d108c585b11cce2a))

## [0.5.4](https://github.com/bruzit/github-actions-and-workflows/compare/v0.5.3...v0.5.4) (2026-08-27)

### Bug Fixes

* **semantic-release:** pin conventional-changelog-writer to v9 ([769db5d](https://github.com/bruzit/github-actions-and-workflows/commit/769db5d4722e6da635e8015e78729e7beb359f27))
* **semantic-release:** pin conventionalcommits preset to v9 ([952bd8b](https://github.com/bruzit/github-actions-and-workflows/commit/952bd8b273e233e2b6e5ed9442916b7f6db152d1))

## [0.5.3](https://github.com/bruzit/github-actions-and-workflows/compare/v0.5.2...v0.5.3) (2026-08-01)

## [0.5.2](https://github.com/bruzit/github-actions-and-workflows/compare/v0.5.1...v0.5.2) (2026-07-26)

## [0.5.1](https://github.com/bruzit/github-actions-and-workflows/compare/v0.5.0...v0.5.1) (2026-07-25)

## [0.5.0](https://github.com/bruzit/github-actions-and-workflows/compare/v0.4.0...v0.5.0) (2026-07-25)

## [0.4.0](https://github.com/bruzit/github-actions-and-workflows/compare/v0.3.0...v0.4.0) (2026-03-29)

### Features

* additional semantic-release plugins ([3baaf53](https://github.com/bruzit/github-actions-and-workflows/commit/3baaf53690083e652e2d6de3219719e1aa966a87))

## [0.3.0](https://github.com/bruzit/github-actions-and-workflows/compare/v0.2.0...v0.3.0) (2026-03-22)

### Features

* add semantic release major tag ([0c99103](https://github.com/bruzit/github-actions-and-workflows/commit/0c9910378ac7738e40655f826e9cf57b770100db))

## [0.2.0](https://github.com/bruzit/github-actions-and-workflows/compare/v0.1.0...v0.2.0) (2026-03-20)

### Features

* make the semantic release workflow reusable ([b9932fb](https://github.com/bruzit/github-actions-and-workflows/commit/b9932fbcb52c01f30dcb550ef442990f52647733))

### Bug Fixes

* release workflow fails on invalid workflow if ([b57b25b](https://github.com/bruzit/github-actions-and-workflows/commit/b57b25ba3fcab75aff6e89537ea6e6276a71ca15))

## [0.1.0](https://github.com/bruzit/github-actions-and-workflows/compare/v0.0.0...v0.1.0) (2026-03-19)

### Features

* add semantic release workflow and configuration ([df87536](https://github.com/bruzit/github-actions-and-workflows/commit/df875360f1ed4a3cf111b72f3270160f5cbccd24))

### Bug Fixes

* release workflow fails on expected dependency lock ([cdf2eb5](https://github.com/bruzit/github-actions-and-workflows/commit/cdf2eb5046d03dade02507eba721c9a22a62a307))
