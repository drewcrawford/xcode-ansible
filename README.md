At times I need to maintain my own macOS/iOS/appleOS CI infrastructure rather than use prebuilt CI like GitHub actions.  I may need to run on attached iOS devices or test on an exact GPU/driver.

There are Chef/Puppet/etc. scripts floating around for maintaining macOS buildfarms, but they are usually designed for large orgs.  

* They install many tools I don't need, for example to achieve parity with a Linux agent, for programming environments I don't use, third-party tools that may have fallen out-of-favor, etc.  This stuff breaks on beta macOS/Xcode, new hardware like M1, or similar situations of keen interest to developers who primarily write code for apple platforms.  
* It also fails to deal with important cases like multiple xcodes and quickly adapting to new releases
* These scripts are not very modular and generally have little flexibility to produce a more minimal buildagent.
* They are not very fast and often reconfigure things that are already configured.

This is the setup I wanted.  It has only the tools I use, broken up into separate roles for flexiblity, populates runner tags with nice values and it's easy to maintain by non-sysadmins.

# Usage

Install ansible on your client machine.  You can get it via homebrew.  You don't need to install anything on the build agent, besides enabling SSH.

Create an `inventory` file with something like

```
[buildservers]
buildserver-hostname.local
[test]
localhost ansible_connection=local
```

Create a `site.yml` with something like

```yaml
---
- name: setup CI
  hosts: buildservers
  vars:
	  # Where to get Xcode.  Since apple downloads are authwalled, the easiest solution
	  # is to host at a random s3 bucket.  No, this is not the one I use, make your own
	  xcode_url: "https://my-xcode-folder.s3.us-east-1.amazonaws.com"

  roles:
	  # Install xcode 12.3 and make it the default
	- role: xcode
	  vars:
		  xcode_version: "12.3"
		  aliases:
			- default
	- role: git-lfs
	- role: github-actions-runner
	  # Configure a github-actions-runner for a repo
	  vars:
			runner_url: "https://github.com/username/my-repo"
			# You generate this token from https://github.com/username/repo/settings/actions/add-new-runner
			# note that you can omit this line if you don't need to reconfigure the runner.
			runner_token: "my-token"
```

See the relevant README files with more info for the options in each section.

Checkout this repository to the `/roles` folder.

Run with 

```
ansible-playbook site.yml -i inventory --ask-become-pass
```

# License

RPL-1.5, a strong copyleft license.  If you improve these scripts you must release them under similar terms.

You don't need to do anything if you configure and use these scripts, even if you use them for commercial purposes.