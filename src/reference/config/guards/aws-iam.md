---
shortTitle: aws-iam
category:
  - Guard
tag:
  - aws-iam
---

# aws-iam Guard

[Available in <ZillaPlus/>](https://www.aklivity.io/products/zilla-plus)
{.zilla-plus-badge .hint-container .info}

Defines a guard with `AWS IAM` support.

Unlike the other guards, `aws-iam` does not verify an incoming credential. Instead, it acquires AWS credentials from a configured source and uses them to mint a short-lived, SigV4-signed authentication token for an outbound connection, such as a Kafka client connecting to Amazon MSK using SASL/IAM.

```yaml {2}
guards:
  my_aws_iam_guard:
    type: aws-iam
    kind: msk
    options:
      region: us-east-1
      credentials:
        source: assume-role
        role: arn:aws:iam::012345678901:role/MyRole
```

::: tip Role-scoped authorization is not supported
Because `aws-iam` does not track roles or scopes for the credentials it acquires, it cannot satisfy role-gated `guarded` entries. Reference it with an empty role list, for example `guarded: { my_aws_iam_guard: [] }`.
:::

## Configuration (\* required)

<!-- @include: ./.partials/store.md -->

### kind\*

> `enum` [ `msk` ]

The token format to mint from the acquired AWS credentials. Currently, only `msk` is supported, which produces a token compatible with Amazon MSK's SASL/IAM mechanism.

### options\*

> `object`

The `aws-iam` specific options.

```yaml
options:
  region: us-east-1
  credentials:
    source: assume-role
    role: arn:aws:iam::012345678901:role/MyRole
  challenge: 30
```

#### options.region

> `string`

AWS region used to construct the MSK endpoint and to sign the token. Also used by the `assume-role` credential source to reach the regional AWS STS endpoint.

#### options.credentials\*

> `object`

The AWS credential source used to authenticate outbound connections.

```yaml
options:
  credentials:
    source: assume-role
    role: arn:aws:iam::012345678901:role/MyRole
```

##### credentials.source\*

> `enum` [ `default` | `instance-profile` | `assume-role` | `env` | `profile` | `static` ]

Where to acquire AWS credentials from:

- `default`: the AWS default credentials provider chain.
- `instance-profile`: the EC2 instance metadata service.
- `assume-role`: an AWS STS `AssumeRole` call against the role given by `credentials.role`.
- `env`: the `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` environment variables.
- `profile`: a named profile given by `credentials.profile` from the local AWS configuration.
- `static`: the literal `credentials.access-key-id` and `credentials.secret-access-key`.

##### credentials.role

> `string`

IAM role ARN to assume via AWS STS. Required when `credentials.source` is `assume-role`.

```yaml
options:
  credentials:
    source: assume-role
    role: arn:aws:iam::012345678901:role/MyRole
```

##### credentials.profile

> `string`

Named AWS profile to load credentials from. Required when `credentials.source` is `profile`.

```yaml
options:
  credentials:
    source: profile
    profile: my-profile
```

##### credentials.access-key-id

> `string`

AWS access key ID. Required when `credentials.source` is `static`.

```yaml
options:
  credentials:
    source: static
    access-key-id: AKIAIOSFODNN7EXAMPLE
    secret-access-key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
```

##### credentials.secret-access-key

> `string`

AWS secret access key. Required when `credentials.source` is `static`.

#### options.challenge

> `integer`

Number of seconds before the acquired credentials expire at which the guard flags the session for re-authentication. Since the minted token is valid for at most `900` seconds regardless of the underlying credential lifetime, this determines how far ahead of that window a connection is challenged to refresh it.

```yaml
options:
  challenge: 30
```
