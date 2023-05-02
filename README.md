# VS Code Python's Issue Triage GitHub Actions

We host our [GitHub Actions](https://help.github.com/en/actions) for VSCode Python related repos here.

Many of these are not specific to VS Code Python, and can be used in other projects by importing the repository like so:

```yml
steps:
  - name: Checkout Actions
    uses: actions/checkout@v2
    with:
      repository: 'microsoft/vscode-triage-github-actions'
      ref: stable # not recommeneded, use the lastest released tag to ensure stability
  - name: Install Actions
    run: npm install --production
  - name: Run Commands
    uses: ./commands
```

Additionally, in `./api`, we have a wrapper around the Octokit instance that can be helpful for developing (and testing!) your own Actions.

*Note:* All Actions must be compiled/packaged into a single output file for deployment. We use [ncc](https://github.com/zeit/ncc) and [husky](https://github.com/typicode/husky) to do this on-commit. Thus committing can take quite a while. If you're making a simple change to non-code files or tests, this can be skipped with the `--no-verify` `git commit` flag.

### Code Layout

The `api` directory contains `api.ts`, which provides an interface for interacting with github issues. This is implemented both by `octokit.ts` and `testbed.ts`. Octokit will talk to github, testbed mimics GitHub locally, to help with writing unit tests.

The `utils` directory contains various commands to help with interacting with GitHub/other services, which do not have a corresponding mocked version. Thus when using these in code that will be unit tested, it is a good idea to manually mock the calls, using `nock` or similar.

The rest of the directories contain three files:
- `index.ts`: This file is the entry point for actions. It should be the only file in the directory to use Action-specific code, such as any imports from `@actions/`. In most cases it should simply gather any required config data, create an `octokit` instance (see `api` section above) and invoke the command. By keeping Action specific code seprate from the rest of the logic, it is easy to extend these commands to run via Apps, or even via webhooks to Azure Funtions or similar.