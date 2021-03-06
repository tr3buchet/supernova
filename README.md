## supernova - use novaclient with multiple nova environments the easy way

You may like *supernova* if you regularly have the following problems:

* You hate trying to source multiple novarc files when using *nova*
* You get your terminals confused and do the wrong things in the wrong nova environment
* You don't like remembering things
* You want to keep sensitive API keys and passwords out of plain text configuration files (see the "Working with keyrings" section toward the end)
* You need to share common skeleton environment variables for *nova* with your teams

If any of these complaints ring true, *supernova* is for you. *supernova* manages multiple nova environments without sourcing novarc's or mucking with environment variables.

![First world problems - nova style](http://lolcdn.mhtx.net/firstworldproblems-multiplenovaenvironments-20120316-072224.jpg)

### Installation

    git clone git://github.com/rackerhacker/supernova.git
    cd supernova
    python setup.py install

### Configuration

For *supernova* to work properly, each environment must be defined in `~/.supernova` (in your user's home directory).  The data in the file is exactly the same as the environment variables which you would normally use when running *nova*.  You can copy/paste from your novarc files directly into configuration sections within `~/.supernova`.

Here's an example of two environments, **production** and **development**:

    [production]
    NOVA_URL=http://production.nova.example.com:8774/v1.1/
    NOVA_VERSION=1.1
    NOVA_USERNAME = jsmith
    NOVA_API_KEY = fd62afe2-4686-469f-9849-ceaa792c55a6
    NOVA_PROJECT_ID = nova-production

    [development]
    NOVA_URL=http://dev.nova.example.com:8774/v1.1/
    NOVA_VERSION=1.1
    NOVA_USERNAME = jsmith
    NOVA_API_KEY = 40318069-6069-4d9f-836d-a46df17fc8d1
    NOVA_PROJECT_ID = nova-production

When you use *supernova*, you'll refer to these environments as **production** and **development**.  Every environment is specified by its configuration header name.

### Usage

    supernova [--debug] [--list] [environment] [novaclient arguments...]

    Options:
    -h, --help   show this help message and exit
    -d, --debug  show novaclient debug output (overrides NOVACLIENT_DEBUG)
    -l, --list   list all configured environments

##### Passing commands to *nova*

For example, if you wanted to list all instances within the **production** environment:

    supernova production list

Show a particular instance's data in the preprod environment:

    supernova preprod show 3edb6dac-5a75-486a-be1b-3b15fd5b4ab0a

The first argument is generally the environment argument and it is expected to be a single word without spaces. Any text after the environment argument is passed directly to *nova*.

##### Debug override

You may optionally pass `--debug` as the first argument (before the environment argument) to inject the `NOVACLIENT_DEBUG=1` option into the process environment to see additional debug information about the requests being made to the API:

    supernova --debug production list

As before, any text after the environment argument is passed directly to *nova*.

##### Listing your configured environments

You can list all of your configured environments by using the `--list` argument.

### Working with keyrings
Due to security policies at certain companies or due to general paranoia, some users may not want API keys or passwords stored in a plaintext *supernova* configuration file.  Luckily, support is now available (via the [keyring](http://pypi.python.org/pypi/keyring) module) for storing any configuration value within your operating system's keychain.  This has been tested on the following platforms:

* Mac: Keychain Access.app
* Linux: gnome-keyring, kwallet

To get started, you'll need to choose an environment and a configuration option.  Here's an example of some data you might not want to keep in plain text:

    supernova-keyring --set production NOVA_API_KEY

**TIP**: If you need to use the same data for multiple environments, you can use a global credential item very easily:

    supernova-keyring --set global MyCompanyLDAPPassword

Once it's stored, you can test a retrieval:

    # Normal, per-environment storage
    supernova-keyring --get production NOVA_API_KEY

    # Global storage
    supernova-keyring --get global MyCompanyLDAPPassword

You'll need to confirm that you want the data from your keychain displayed in plain text (to hopefully thwart shoulder surfers).

Once you've stored your sensitive data, simply adjust your *supernova* configuration file:

    #NOVA_API_KEY = really_sensitive_api_key_here
    
    # If using storage per environment
    NOVA_API_KEY = USE_KEYRING
    
    # If using global storage
    NOVA_API_KEY = USE_KEYRING['MyCompanyLDAPPassword']

When *supernova* reads your configuration file and spots a value of `USE_KEYRING`, it will look for credentials stored under `NOVA_API_KEY` for that environment automatically.  If your keyring doesn't have a corresponding credential, you'll get an exception.

#### A brief note about environment variables

*supernova* will only replace and/or append environment variables to the already present variables for the duration of the *nova* execution. If you have `NOVA_USERNAME` set outside the script, it won't be used in the script since the script will pull data from `~/.supernova` and use it to run *nova*. In addition, any variables which are set prior to running *supernova* will be left unaltered when the script exits.
