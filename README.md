# aws-sso-creds

`aws-sso-creds` is a helper utility to retrieve [temporary credentials](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp.html) when using [AWS SSO](https://aws.amazon.com/single-sign-on/)

## About

If you're using AWS SSO, you're able to set up your AWS profile like so:

```
[profile sso-profile]
output = json
region = us-west-2
sso_account_id = <my-account-id>
sso_region = us-west-2
sso_role_name = <role-to-assume>
sso_start_url = <special-sso-url>
```

This is great, because it means you're able to login very easily using `aws sso login` from the [AWS CLI](https://aws.amazon.com/cli/)

This retrieves a set of cached credentials, which are saved into `~/.aws/sso/cache` and you can now use the AWS CLI with those credentials.

_However_

Unfortunately, the AWS SDK's in nearly every language currently do not support these credentials. In this case, you can [retrieve temporary credentials](https://aws.amazon.com/blogs/security/aws-single-sign-on-now-enables-command-line-interface-access-for-aws-accounts-using-corporate-credentials/) that look like the AWS credentials you're used to:

```
AWS_ACCESS_KEY_ID=<key>
AWS_SECRET_ACCESS_KEY=<key>
AWS_SESSION_TOKEN=<key>
```

However, it's really quite annoying to have to login to the URL and grab these tokens manually. The AWS CLI has support for retrieving them, but you have to run:

```bash
aws sso get-role-credentials --role-name <SOME_ROLE_I_CANNOT_REMEMBER> --account-id <WHATS_MY_ACCOUNT_ID_AGAIN?> --access-token <I_HAVE_TO_LOOK_THIS_UP_IN_A_FILE_WHERE?>
```

This simple utility is designed to take the pain out of this process. It can:

- Grab you a set of credentials to copy and paste for a specific account/profile (If you're so inclinded)
- Generate an `eval` compatible output to ease the process of grabbing these credentials
- List the accounts and roles you have access to for ease of management

# Usage

## Get credentials

If you just want to retrieve a set of credentials for your AWS SSO based profile, just run `aws-sso-creds get`:

```bash
$ aws-sso-creds get
Your temporary credentials for account <foo> are:

AWS_ACCESS_KEY_ID	 <KEY>
AWS_SECRET_ACCESS_KEY <ACCESS_KEY>
AWS_SESSION_TOKEN	<A_LONG_SESSION_TOKEN>

These credentials will expire at: Mon Oct 31 16:03:20 PST 52495
```

`aws-sso-creds` will automatically use the `AWS_PROFILE` environment variable you have set. You can also specify a profile with `aws-sso-creds --profile`

## Populate your shell with vars

If you want to just get going without any copying and pasting, use [eval](https://man7.org/linux/man-pages/man1/eval.1p.html) with `aws-sso-creds export`

```bash
eval $(aws-sso-creds export)
```

This command generates output in the form of `export` variables:

```bash
$ aws-sso-creds export
export AWS_ACCESS_KEY_ID=<KEY>
export AWS_SECRET_ACCESS_KEY=<SECRET_KEY>
export AWS_SESSION_TOKEN=<SESSION_TOKEN>
```

## List accounts

You can also list the accounts you have available within AWS SSO:

```bash
$ aws-sso-creds list accounts
ID             NAME                 EMAIL ADDRESS
<id>           dev-sandbox          aws-contact+dev-sandbox@example.com
<id>           -ci                  aws-contact+ci@example.com
```

## List account roles

You can list the roles available in an account like so:

```
$ aws-sso-creds list roles <account-id>
```

_NOTE:_ currently this tool doesn't support multiple roles when getting credentials, if this is necessary, please file a feature request

## Save into AWS credentials and config files

```
$ aws-sso-creds set <non-SSO profile> -p <SSO profile>
```

# Installation

## For Go users

Assuming you've already added $GOPATH/bin to your $PATH: `export PATH="$PATH:$GOPATH/bin"`

```
go install github.com/thuannfq/aws-sso-creds/cmd/aws-sso-creds@latest
```
