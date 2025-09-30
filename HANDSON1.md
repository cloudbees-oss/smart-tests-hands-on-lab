# Lab 1. Environment setup

First, locally clone the repositories you want to try Smart Tests with.
If your test and production code reside in two different repositories, clone both of them.

If your production code are split among multiple repositories, we recommend you clone a couple of major ones, just to keep this workshop manageable.

## Install Launchable command

You interact with Smart Tests using a command line tool called `launchable`.

You can install it with pip:

```
pip3 install --upgrade launchable~=1.0
```

> [!TIP]
> If you encounter `externally managed error`
> <details>
>
> Try installing `pipx` and use it.
> see: https://pipx.pypa.io/stable/installation/
>
> ```
> pipx install launchable~=1.0
> ```
> </details>


Let’s check that it’s installed correctly:

```
launchable --help
```

>[!TIP]
> If `launchable` is not found on your `PATH`
> <details>
> Run the following command to find out where `pip3` installed the script:
>
> ```
> pip3 show --files launchable | grep -E 'bin/launchable$|^Location'
> ```
>
> This command will produce output like this:
>
> ```
> Location: /home/kohsuke/anaconda3/lib/python3.9/site-packages
>   ../../../bin/launchable
> ```
>
> Concatenate two paths to obtain the location, in the example above, that'd be `/home/kohsuke/anaconda3/lib/python3.9/site-packages/../../../bin/launchable`, which is `/home/kohsuke/anaconda3/bin/launchable`
>
> Add the directory portion of this to `PATH` by trimming the trailing `launchable`, like this:
>
> ```
> export PATH=/home/kohsuke/anaconda3/bin:$PATH
> ```

>[!TIP]
> Alternatively, you can use the `launchable` command in a Docker container: `docker run --rm cloudbees/launchable --help`


## Obtain an API token

You need an API token to use Smart Test.
Go to your workspace’s Settings > API Token and generate a new token.

**Node:** If you haven’t created a workspace yet, please refer to `SIGN_UP.md` to set one up.

<img src="https://github.com/user-attachments/assets/1f17be96-acf9-4825-8f9f-06790a14dc1c" width="50%">

<br>

Click the **Create a new API key**

<img src="https://user-images.githubusercontent.com/536667/191438711-b15eb234-e3d5-4ba2-b2fb-11d0ebd92d18.png" width="50%">

Click **Copy** key and copy API key.

<img src="https://github.com/user-attachments/assets/5025328b-fc20-4eb1-b7f2-346aab60e013" width="50%">

The `launchable` command expects an API token to be set in the `LAUNCHABLE_TOKEN` environment variable.

```sh
export LAUNCHABLE_TOKEN=<API TOKEN>
```


## Make sure everything is in order

`launchable verify` command is a convenient way to make sure all the prerequisites are met and the API key is valid:

```
launchable verify
```

If you see a message like this, you’re all set:

```
Organization: 'organization'
Workspace: 'workspace'
Proxy: None
Platform: 'Linux-6.10.14-linuxkit-aarch64-with-glibc2.36'
Python version: '3.11.13'
Java command: 'java'
launchable version: '1.106.2'
Your CLI configuration is successfully verified 🎉
```

___

If you see the help message, the installation was successful.
You can now move on to [the next step](HANDSON2.md).



