
# GitHub Action Variables

<img src="06-gha-variables_files/figure-html//1x0Cnk2Wcsg8HYkmXnXo_0PxmYCxAwzVrUQzb8DUDvTA_g290614d43ec_0_52.png" width="100%" style="display: block; margin: auto;" />

The GitHub Actions environments have variables that are already set by default in the environment but you can also set environment variables yourself.

### Types of variables

There are two types of variables in GitHub Actions.

1. `Default` - Ones GitHub already sets for you.
2. `User set` - Ones you set yourself.

To print things out, you can use this kind of notation in bash or other contexts in the yaml file.
```
echo ${{ github.repository }}
```

In this next exercise we'll explore different ways to use variables.

## Exercise 3 - Exploring Variables

For this exercise, we are going to continue to use the example repository that we set up in the previous chapter.

1. Create a new branch to work from.

<img src="06-gha-variables_files/figure-html//1x0Cnk2Wcsg8HYkmXnXo_0PxmYCxAwzVrUQzb8DUDvTA_g2903858106e_0_4.png" width="100%" style="display: block; margin: auto;" />
From command line:
```
`git checkout -b "env-var"`
```

2. For this exercise we are going to copy over another GHA yaml to explore. This time, move the `exploring-var-and-secrets.yml` file to your `.github/workflows` directories you made in the previous chapter.

<img src="06-gha-variables_files/figure-html//1x0Cnk2Wcsg8HYkmXnXo_0PxmYCxAwzVrUQzb8DUDvTA_g2903858106e_0_17.png" width="100%" style="display: block; margin: auto;" />
From command line:
```
mv activity-1-sample-github-actions/exploring-var-and-secrets.yml .github/workflows/exploring-var-and-secrets.yml
```

3. Now follow the same set of steps we used in the previous chapter to Add, Commit, Push the changes.

<img src="06-gha-variables_files/figure-html//1x0Cnk2Wcsg8HYkmXnXo_0PxmYCxAwzVrUQzb8DUDvTA_g2903858106e_0_25.png" width="100%" style="display: block; margin: auto;" />
From command line:
```
git add .github/*
git commit -m "exploring gha variables"
git push --set-upstream origin env-var
```

4. Now create a pull request with the changes you just made. (Refer to the previous chapter if you need reminders on how to do this).

5. On your pull request page on GitHub, click on the `Details` button next to your workflow run. Keep this handy because we will dive into the details of what we just ran.

<img src="06-gha-variables_files/figure-html//1x0Cnk2Wcsg8HYkmXnXo_0PxmYCxAwzVrUQzb8DUDvTA_g280d2b56f79_0_3250.png" width="100%" style="display: block; margin: auto;" />

### Default variables

You can read [the latest documentation about GitHub Action default variables here](https://docs.github.com/en/actions/learn-github-actions/variables). But here's some highlights.

| name | example output | explanation |
|------|----------------|-------------|
|GITHUB_REPOSITORY| username/repository_name | This prints out what repo this is run from|
|GITHUB_REF| refs/pull/1/merge | The branch or tag that triggered this workflow. But note that this will be blank if the trigger is not based or related to branches or tags. For example a `workflow_dispatch` wouldn't have this|
|GITHUB_ACTOR| cansavvy | The GitHub handle of the person who caused this workflow to run|


Below shows an example of the log of the  where we printed out these default GitHub variables.

<img src="06-gha-variables_files/figure-html//1x0Cnk2Wcsg8HYkmXnXo_0PxmYCxAwzVrUQzb8DUDvTA_g2903858106e_0_53.png" width="100%" style="display: block; margin: auto;" />

### User set variables

#### env:

There are different ways to set variables. The simplest way to set variables is within a step you can set them using `env:`. Underneath `env:` you write the `name` of the variable on one side of the colon and then the definition on the other side. For example, in our yaml file we had:

```
- name: Hello, but make it personal
  run: echo "Hello $First_Name."
  env:
    First_Name: Candace
```

This set up printed out `Hello Candace` in the logs as our output.

This might be useful, but if we want an environmental variable to be stored and retrieved between steps we'll need to use something different.

#### Setting output variables

If we'd like one step to be able to retrieve information from another step we'll need to send a variable to the GITHUB_OUTPUT.

To do this we can use this sort of set up:


<img src="06-gha-variables_files/figure-html//1x0Cnk2Wcsg8HYkmXnXo_0PxmYCxAwzVrUQzb8DUDvTA_g290614d43ec_0_2.png" width="100%" style="display: block; margin: auto;" />

Step that sets a variable depending on some output:

```
# How to export a variable to a next step
- name: Setting output to the environment at large
  id: step_name
  run: echo "results=5" >> $GITHUB_OUTPUT
```
Here we are naming the variable `results` and the notation `>> $GITHUB_OUTPUT` is always there.

In this example, `results` is only set equal to `5` but you could see how this might be made to be more complicated. Like perhaps the results are a bash command output like:
```
"time=$(date +'%Y-%m-%d')" >> $GITHUB_OUTPUT
```
This would allow us to have a time stamp of when this step was run.

Or perhaps we are running a script that outputs a result:
```
results=$(Rscript utils/script.R)
```

#### Using output variables

To use this output variable in a subsequent step we have to use this kind of setup:

`steps.step_name.outputs.results` where `step_name` is the `id: ` we set for the step that set this variable (see above) and `results` is the name of the variable we set. And, as is typical we need the `${{ }}` notation.

```
# How to print out the variable we just saved
- name: Print out that variable in a later step
  run: echo ${{ steps.step_name.outputs.results > 3 }}
```

This is nifty because now we can use the result of one step to determine whether or not we run a subsequent step. GitHub Action steps can have conditional or `if` statements.

Maybe we only want a step to run if the result is something specific:
```
- name: Conditional step
  # Here we are only going to do this step if the results from the previous step are bigger than 3
  if: ${{ steps.step_name.outputs.results > 3 }}
  run: echo 'the results are greater than 3!'
```

Or, maybe we want to make sure the whole workflow shuts down if a variable is something in particular like this example below.

```
- name: Shut it down
  # Here we are only going to do this step if the results from the previous step are bigger than 3
  if: ${{ steps.step_name.outputs.results =< 3 }}
  run: |
    echo 'the results are less than or equal to 3! -- going to exit!'
    exit 1

```

#### Setting and grabbing secrets

What if the string or variable we need is not something we can supply in the YAML itself? Perhaps we have credentials or something that cannot be shared publicly but that we need it to complete our steps. That's where GitHub secrets come in handy!

[Read more about GitHub secrets here.](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions).

One very common type of GitHub secret you may need to add is a GitHub Personal Access Token (sometimes abbreviated PAT). A personal access token is a string set that, when provided, gives access to a user's GitHub account. [Read more about tokens here](https://github.com/settings/tokens).

For GitHub actions that are doing things that require authorization or particular permissions levels, you will need to provide your GitHub action with your personal access token (PAT) that you store as a GitHub secret.

### Activity: Setting GitHub secrets

Let's practice this by setting a GitHub Access token as a secret!

#### Make a Personal Access token

You can store any alphanumeric string as your GitHub secret. It may be an API key or authorization keys from some other software program. But for this example, we will use an authorization key for GitHub.

Recall we have may have to give authorization to a GitHub action some times, because we are not actually running this with our user account, this job is being sent to GitHub for them to run on their servers somewhere.

1. First make your own personal access token by going here: https://github.com/settings/tokens You can find this page by going to your own profile, and then to `Settings` and `Developer settings`.

The GitHub Documentation for how to make PATs is here: https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens
But we'll walk through it together now.

2. Underneath `Tokens (classic)` click `Generate new token` and pick `Generate new token (classic)`.
You will likely have to enter your password at this point.

3. Underneath `Note` write something that will remind you about where you are using this PAT. Check the `repo` workflow. (Depending on what you are trying to do you may have to check other boxes but for a lot of the permissions you'll need `repo` will do).

4. Scroll to the bottom of the page and click `Generate`. Your token will be shown on the next page. You'll keep this handy because they won't show it to you again. Be careful not to share this any place publicly because it will give someone authorization to you GitHub account!

#### Creating a GitHub Secret

1. Return to your repository that we were using for these activities.
2. `Settings` > `Secrets and variables` > `Actions` > `New Repository Secret`.
3. Name your secret something. In this example, let's call it `GH_PAT`. You'll want to name your secret something that relates to what it is.
4. Now copy and paste in the secret section.

<img src="06-gha-variables_files/figure-html//1x0Cnk2Wcsg8HYkmXnXo_0PxmYCxAwzVrUQzb8DUDvTA_g290614d43ec_0_38.png" width="100%" style="display: block; margin: auto;" />

##### Referencing a GitHub secret in a GitHub action

To retrieve a GitHub secret amidst a GitHub Action workflow run, you do this sort of notation: `${{ secrets.SECRET_NAME}}` Where `SECRET_NAME` directly is the name you used for your GitHub secret.
```
# Here's how we'd reference a secret
- name: How do we reference a GITHUB secret?
  run: ${ secrets.SECRET_NAME }
```
In the previous step we named our secret `GH_PAT` so if we needed to use it in our workflow we would use `${ secrets.GH_PAT }`.

Perhaps at this point you are worried that your logs may accidentally display your GitHub secret if you did something like:

```
run: echo ${ secrets.GH_PAT }
```

But, you don't have to worry about that part, in your logs your secrets will show as `***` and will not be displayed.

##### Activity: Using a GitHub secret

1. On your repository, go to your `01-exploring-var-and-secrets.yml` file from your working branch.
2. Click the `edit this file` button.
3. Scroll to the bottom. Uncomment the last step step.
It should look like this:
```
# Here's how we'd reference a secret
- name: How do we reference a GITHUB secret?
  run: ${ secrets.SECRET_NAME }
```
4. Replace `SECRET_NAME` with what you named your secret (probably `GH_PAT`).
5. Commit that change to your file.
6. Push that change to your file.  
7. Take a look at the log by clicking `Details`.

<img src="06-gha-variables_files/figure-html//1x0Cnk2Wcsg8HYkmXnXo_0PxmYCxAwzVrUQzb8DUDvTA_g280d2b56f79_0_3358.png" width="100%" style="display: block; margin: auto;" />

What you should see is that the workflow runs again, tries to print the GitHub secret out but really just shows a `***`.
