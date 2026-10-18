# CPR RDS compatibility image

This build pins upstream CPR 3.29.2 at commit
`2221f400da29731c8f62464dd097945b85cf82f1` and changes only PostgreSQL
password validation plus its configuration tests. PostgreSQL credentials accept
16–128 bytes without control characters or whitespace-only values. The existing
48-character hexadecimal credentials remain valid. Redis keeps its upstream
48-character hexadecimal requirement. Database passwords embedded in URLs remain
rejected and debug redaction is unchanged.

Huawei RDS rejected the upstream 48-character password under its enhanced
8–32-character password policy. This application patch allows a strong random
32-character RDS password without changing the shared database security policy.

The workflow runs formatting checks, configuration tests, and gateway-store
Clippy before building the release binary. The runtime is the immutable upstream
amd64 image, with only the binary and version labels replaced. No production
credentials or data are used in CI. Publishing the image does not deploy it.

Run the manual `Build CPR RDS compatibility image` workflow. Record its resulting
ACR digest and perform isolated RDS restore/migration and failback tests before
deployment. The custom tag is `3.29.2-rds.1`; it is not an upstream release.
