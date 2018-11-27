# cookiecutter-ansible-role-kitchen

This cookiecutter generates a project skeleton for developping ansible roles
using test-kitchen, docker and inspec for testing. This is the fastest, most
stable setup for practicing TDI with Ansible in my experience.

This cookiecutter does not include Windows support by default, although all of
Ansible, test-kitchen and inspec support Windows to some degree. That said,
unless you are running Windows yourself you will probably have to use Vagrant
instead of Docker.

# Requirements
- docker
- make
- ruby with the bundler gem.
- python with the cookiecutter and pipenv packages.

# Installation

```bash
# I usually start with a previously created git repo containing at least a
# license file.
git clone https://github.com/esoucy19/ansible-role-your-role
cd my-role
cookiecutter gh:esoucy19/cookiecutter-ansible-role-kitchen

# Answer the prompts; here we'll use the default "your-role" name.
# This puts the files in a subdirectory, which is probably not what you want.
mv your-role/{.,}* ./
rm -r your-role

# Install the gems and packages:
make install

# The cookiecutter contains a dummy test to verify everything is working:
make test
```

# Features

- Fully functional kitchen config using centos/systemd as a base image.
- Automated linting checks using ansible-lint, yamllint and rubocop.
- Gemfile and Pipfile included for easy setup.
- Makefile included to run common commands:
  - make install: Run bundle install and pipenv install.
  - make lint: Run linters on project.
  - make converge, verify, test, etc: Run usual kitchen commands.
- Project folder is symlinked in roles using the actual role name. That way you
  don't have to call the role using your repo name in your tests.
- Travis-ci support to automatically test your role and get the much-coveted
"build passing" badge on Galaxy.

# Additional information

## Galaxy and travis integration

To setup for automatic integration with travis-ci, first enable your git repo in
your travis account. Go to your account settings and copy your travis token to
the clipboard. Then run the following commands in the root of your repository:

```bash
ansible-galaxy login

# Use your own github user, repo and travis token here
ansible-galaxy setup travis github_user github_repo travis_token
```

And that's it. Now every push to your repository will trigger a travis build,
which will then trigger an import in Ansible Galaxy. If all goes well, your role
will now bear a nice "build passing" badge. Use your newfound powers wisely!

## Docker gotchas

### SSHD

This setup is meant to work through ssh exactly like ansible normally does. This
means the guest system must have sshd running. Normal Docker practice would be
to run a single process per container, but here we are cheating and abusing
Docker to try and make it simulate a true VM. This is why we are using the
centos/systemd image as our base image in the default configuration.

Kitchen-docker also needs some options set in sshd to properly connect to the
container. We are appending those to /etc/sysconfig/sshd in our
provision_command section. If you need to add more platforms to your role, make
sure to copy those commands or include an equivalent.

### Package doc files

The base centos image (and most docker images I presume) has the package manager
configured so it will not download documentation for the packages it installs.
This is normally fine, but if you ever come accross a package that puts config
files in its docs (I'm looking at you, Zabbix!), then you will want to instruct
the package manager to download package docs.

For yum, this is done in /etc/yum.conf, through this little line of code:

```bash
tsflags=nodocs
```

This line is normally not present in stock CentOS7, so we know it was added by
the image developpers. An easy way to remove this line would be to add a
pre_tasks section to playbook.yml.

```yaml
- hosts: all
  pre_tasks:
    - name: Ensure yum downloads package docs
      lineinfile:
        path: /etc/yum.conf
        regexp: '^tsflags=nodocs'
        state: absent
  roles:
    - your-role
```

If you are making a container-enabled role, consider baking this task into
your role itself. Might as well then have a task to install the specific package
that requires its docs, and a third task to re-add the nodocs configuration.
That way you are only downloading the docs for a single package and your
containers will stay lean and happy.

### Missing net-tools

Docker containers are minimalist (duh), so even basic packages like netstat
or ss won't be present by default. It just so happens that inspec's port
resource requires ss to be installed and will fail with a pretty unhelpful
error message if it isn't. This is why we install net-tools ourselves in the
provision_command section. Make sure you do the same if you add other platforms,
or if you end up using inspec resources that require additional packages to
be installed.

# License

MIT
