# Security Policy

## Supported Versions

Security updates are provided for the following versions of LunarAST specifications and tooling:

| Component | Version | Supported |
|:---|:---|:---|
| Ecosystem Mother Specification | 1.5 | ✅ |
| RouteAST Sub-Protocol | 0.6.0 | ✅ |
| `lunar` CLI | Latest release | ✅ |
| `lunar-gateway` | Latest release | ✅ |

## Reporting a Vulnerability

**Please do not report security vulnerabilities through public GitHub Issues.**

Instead, please report them via email to **[INSERT SECURITY EMAIL]**.

You should receive a response within 48 hours. If the issue is confirmed, we will release a patch as soon as possible depending on complexity.

## Security Model Reference

LunarAST implements the following defense-in-depth measures (see Mother Specification, Section 11):

1. **Injection Attack Prevention**: Lexical filtering of malformed magic comments (`// lunar:`) using language-specific regex patterns combined with AST node type verification.
2. **Key Rotation Overflow Defense**: Maximum of 3 public keys retained per logical project (`MAX_KEYS_PER_REPO = 3`).
3. **S3 Credential Minimization**: CI Actions hold only `s3:PutObject` and `s3:GetObject` permissions.
4. **Destructive Action Mistouch Prevention**: All destructive CLI actions require secondary interactive confirmation.

## Vulnerability Disclosure

We follow a coordinated disclosure process. Security advisories will be published via GitHub Security Advisories once a fix is available.

## Scope

The following are within scope for our security program:

- Vulnerabilities in the `lunar` CLI tool
- Vulnerabilities in the `lunar-gateway` edge gateway
- Protocol-level flaws that could lead to data leakage or unauthorized access
- JWT signature verification bypasses

The following are **out of scope**:

- Vulnerabilities in third-party adapters not maintained by the LunarAST organization
- Issues in user-deployed configurations that do not follow the specification's security recommendations
