# Composite Retry

Retries an Github Action step or command on failure.

Works with either shell commands or other actions to retry.

This package is very similar and inspired by [retry-action](https://github.com/marketplace/actions/retry-action)
but is built as a [Composite Action](https://docs.github.com/en/actions/creating-actions/about-custom-actions#composite-actions)
instead of a [JavaScript action](https://docs.github.com/en/actions/creating-actions/about-custom-actions#javascript-actions)
to avoid some of the complexity and nuanced issues that come with JavaScript actions.

# Why

Github actions which use an Internet connection can fail when connection is lost :

```
Run actions/setup-node@v1
connect ETIMEDOUT 104.20.22.46:443
Waiting 15 seconds before trying again
connect ETIMEDOUT 104.20.22.46:443
Waiting 18 seconds before trying again
Error: connect ETIMEDOUT 104.20.22.46:443
```

It is a cause of failed jobs. For this case, the action rudolph9/composite-retry can retry the action immediately after fail or with some delay. And if the connection will be restored, then the job will continue the normal run.

# Background

I was trying to use [retry-action](https://github.com/marketplace/actions/retry-action) to retry
[docker/build-push-action](https://github.com/marketplace/actions/build-and-push-docker-images) because
of [this bug](https://github.com/docker/build-push-action/issues/761) but ran into issues accessing the
docker files in the repo:
`buildx failed with: ERROR: could not find '/home/runner/work/repo-name/jobs/builds: stat '/home/runner/work/repo-name/jobs/builds: no such file or directory`

After some digging I found a very verbose way to achieve "retries" using [`continue-on-error`](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstepscontinue-on-error) and copy and pasting the step to be retried conditionally executing using [`if` expressions](https://docs.github.com/en/actions/learn-github-actions/expressions) checking the [`outcome`](https://docs.github.com/en/actions/learn-github-actions/contexts#steps-context) of the step we're retrying.

e.g.:
```
  steps:
    - id: foo
      name: Foo Action
      uses: foo/action@v3
      with:
        bar: ./
      continue-on-error: true
    - id: foo-retry
      name: Foo Action
      if: ${{ steps.foo.outcome == 'failure' }}
      uses: foo/action@v3
      with:
        bar: ./
```

This works really well! But this is cumbersome and not DRY so I build this Action.
