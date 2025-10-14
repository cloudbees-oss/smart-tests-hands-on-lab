# Lab 0. Prerequisites 

The following are prerequisites for this workshop:

1. Look into your internal infosec policies and make sure you can let CloudBees process your source code during this workshop. Note we do not store your source code, nor are they used to train any AI models. See [data privacy and protection policy](https://www.launchableinc.com/docs/resources/policies/data-privacy-and-protection/) for more details.

1. Provide recent test results data from your actual project. This allows us to personalize your workshop experience by pre-populating Smart Tests dashboards and highlighting insights such as _Unhealthy tests_, _Trends_ and _AI-based failure triage_.
   - Provide **atleast 6 recent runs** worth of test results (ideally spanning 1–2 weeks)
   - Include runs with both **passing and failing tests**
   - Make sure `stdout` and `stderr` logs are captured in your reports

1. You need a computer with `git`, `python3`, and `java` installed.

1. Prepare a repository that contains test code where you want to try PTS. We recommend using a repository you normally work with. (You will not need to push any code during the hands-on.)

1. Share your GitHub ID with us, so that we can add you as a collaborator to the workshop.

1. Follow the invitation link we send to you, and perform the sign-up process.

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
