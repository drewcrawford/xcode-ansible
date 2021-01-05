# Drew's xcode role

This installs xcode suitable for development on a clean mac.

# Requirements

```bash
ansible-galaxy collection install community.general
```

# Usage

## `xcode_url`
Generally, your global vars should contain a path to download Xcode.  Apple requires authentication
for downloads, which is no good, so you should host somewhere else (such as Amazon S3).

```yaml
hosts: buildservers
  vars:
	  xcode_url: "https://sorry-dont-steal-my-bandwidth.amazonaws.com"
```

## `xcode_version`

This ought to be a role-specific var, so that you can use the role multiple times to install various xcodes.  The value ought to uniquely identify a specific binary, e.g. including beta numbers if applicable.

roles:
- role: xcode
  vars:
	  xcode_version: "12.2"

We will try to download a file like `Xcode_{{xcode_version}}.xip` from the `xcode_url` directory.  This follows apple's naming scheme (e.g., files like `Xcode_12.2.xip`)

The xcode will be installed to `/Applications/Xcode_{{xcode_version}}.app`.

## aliases

This is a list of aliases (alternative names) for this xcode.  For each item in the list,

1.  We will symlink the xcode to /Applications/Xcode_{{alias}}.app.  In this way you can install `Xcode_12.3b2` and make it available to scripts wanting `Xcode_12.3` or `Xcode_12`.
2.  We will add the xcode to `programmatic_labels`.  Ultimately the `github-actions-runner` will use this to create labels for the runner.

The special alias `default` is handled separately.  This indicates the xcode should be symlinked to `/Applications/Xcode.app`, and runner will be given the 'xcode' label.

## details

We perform the following

1.  Install xcode
2.  Deal with EULA
3.   enable debugging on the machine.  In practice, this means the current user owns the machine (if you can attach a debugger you can do just about anything).  Keep this in mind for security implications.
4.  Install xcpretty
5.  Symlink aliases as described, if applicable
6.  Generate values for labels, which can be used by other roles

