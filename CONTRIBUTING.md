# Contributing

Thanks for taking the time to contribute to this project!

## Code of conduct

This project adheres to the [Contributor Covenant](https://www.contributor-covenant.org/version/2/1/code_of_conduct/):

We as members, contributors, and leaders pledge to make participation in our community a harassment-free experience for everyone, regardless of age, body size, visible or invisible disability, ethnicity, sex characteristics, gender identity and expression, level of experience, education, socio-economic status, nationality, personal appearance, race, caste, color, religion, or sexual identity and orientation.

## Releases

Releases are not automated yet. For now it's a manual process.

To publish a new release, run the following in your local checkout:

```sh
git checkout main
git fetch --all --tags
git tag vX.Y.Z    # specific SemVer version
git tag vX        # vague major version identifier for use workflow files
git push --tags
```

Then visit [github.com/replicate/setup-cog/tags](https://github.com/replicate/setup-cog/tags) to see the tags.
