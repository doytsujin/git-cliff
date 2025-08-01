---
sidebar_position: 1
---

# Context

Context is the model that holds the required data for a template rendering. The [JSON](https://en.wikipedia.org/wiki/JSON) format is used in the following examples for the representation of a context.

## Conventional Commits

> conventional_commits = **true**

For a [conventional commit](/docs/configuration/git#conventional_commits) like this,

```
<type>[scope]: <description>

[body]

[footer(s)]
```

following context is generated to use for templating:

```json
{
  "version": "v0.1.0-rc.21",
  "message": "The annotated tag message for the release"
  "commits": [
    {
      "id": "e795460c9bb7275294d1fa53a9d73258fb51eb10",
      "group": "<type> (overridden by commit_parsers)",
      "scope": "[scope]",
      "message": "<description>",
      "body": "[body]",
      "footers": [
        {
          "token": "<name of the footer, such as 'Signed-off-by'>",
          "separator": "<the separator between the token and value, such as ':'>",
          "value": "<the value following the separator",
          "breaking": false
        }
      ],
      "breaking_description": "<description>",
      "breaking": false,
      "conventional": true,
      "merge_commit": false,
      "links": [
        { "text": "(set by link_parsers)", "href": "(set by link_parsers)" }
      ],
      "author": {
        "name": "User Name",
        "email": "user.email@example.com",
        "timestamp": 1660330071
      },
      "committer": {
        "name": "User Name",
        "email": "user.email@example.com",
        "timestamp": 1660330071
      },
      "raw_message": "<type>[scope]: <description>\n[body]\n[footer(s)]"
    }
  ],
  "commit_id": "a440c6eb26404be4877b7e3ad592bfaa5d4eb210 (release commit)",
  "timestamp": 1625169301,
  "repository": "/path/to/repository",
  "commit_range": {
      "from": "(id of the first commit used for this release)",
      "to": "(id of the last commit used for this release)"
  "submodule_commits": {
    "<relative/submodule_path/in/repository>": [
      {
        "id": "c10d0f12e85975bc1e8f41eed693c58eca1894eb",
        "message": "release (#1671)",
        ...
      },
    ],
  },
  "statistics": {
    "commit_count": 1,
    "commits_timespan": 0,
    "conventional_commit_count": 1,
    "links": [
      {
        "text": "#452",
        "href": "https://github.com/orhun/git-cliff/issues/452",
        "count": 1
      }
    ],
    "days_passed_since_last_release": 0
  },
  "previous": {
    "version": "previous release"
  }
}
```

:::info

See the [GitHub integration](/docs/integration/github) for the additional values you can use in the template.

:::

### Footers

A conventional commit's body may end with any number of structured key-value pairs known as [footers](https://www.conventionalcommits.org/en/v1.0.0/#specification). These consist of a string token naming the footer, a separator (which is either `: ` or ` #`), and a value, similar to [the git trailers convention](https://git-scm.com/docs/git-interpret-trailers).

For example:

- `Signed-off-by: User Name <user.email@example.com>`
- `Reviewed-by: User Name <user.email@example.com>`
- `Fixes #1234`
- `BREAKING CHANGE: breaking change description`

When a conventional commit contains footers, the footers are passed to the template in a `footers` array in the commit object. Each footer is represented by an object with the following fields:

- `token`, the name of the footer (preceding the separator character)
- `separator`, the footer's separator string (either `: ` or ` #`)
- `value`, the value following the separator character
- `breaking`, which is `true` if this is a `BREAKING CHANGE:` footer, and `false` otherwise

### Breaking Changes

`breaking` flag is set to `true` when the commit has an exclamation mark after the commit type and scope, e.g.:

```
feat(scope)!: this is a breaking change
```

Or when the `BREAKING CHANGE:` footer is defined:

```
feat: add xyz

BREAKING CHANGE: this is a breaking change
```

`breaking_description` is set to the explanation of the breaking change. This description is expected to be present in the `BREAKING CHANGE` footer. However, if it's not provided, the `message` is expected to describe the breaking change.

If the `BREAKING CHANGE:` footer is present, the footer will also be included in
`commit.footers`.

Breaking changes will not be skipped if [`protect_breaking_commits`](/docs/configuration/git#protect_breaking_commits) is set to `true`, even when matched by a skipping [commit_parser](/docs/configuration/git#commit_parsers).

### Committer vs Author

From [Git docs](https://git-scm.com/book/en/v2/Git-Basics-Viewing-the-Commit-History):

> You may be wondering what the difference is between author and committer. The author is the person who originally wrote the work, whereas the committer is the person who last applied the work. So, if you send in a patch to a project and one of the core members applies the patch, both of you get credit — you as the author, and the core member as the committer.

## Non-Conventional Commits

> conventional_commits = **false**

If [`conventional_commits`](/docs/configuration/git#conventional_commits) is set to `false`, then some of the fields are omitted from the context or squashed into the `message` field:

```json
{
  "version": "v0.1.0-rc.21",
  "message": "The annotated tag message for the release"
  "commits": [
    {
      "id": "e795460c9bb7275294d1fa53a9d73258fb51eb10",
      "group": "(overridden by commit_parsers)",
      "scope": "(overridden by commit_parsers)",
      "message": "(full commit message including description, footers, etc.)",
      "conventional": false,
      "merge_commit": false,
      "links": [
        { "text": "(set by link_parsers)", "href": "(set by link_parsers)" }
      ],
      "author": {
        "name": "User Name",
        "email": "user.email@example.com",
        "timestamp": 1660330071
      },
      "committer": {
        "name": "User Name",
        "email": "user.email@example.com",
        "timestamp": 1660330071
      },
      "raw_message": "(full commit message including description, footers, etc.)"
    }
  ],
  "commit_id": "a440c6eb26404be4877b7e3ad592bfaa5d4eb210 (release commit)",
  "timestamp": 1625169301,
  "repository": "/path/to/repository",
  "commit_range": {
      "from": "(id of the first commit used for this release)",
      "to": "(id of the last commit used for this release)",
  },
  "statistics": {
    "commit_count": 1,
    "commits_timespan": 0,
    "conventional_commit_count": 0,
    "links": [
      {
        "text": "#452",
        "href": "https://github.com/orhun/git-cliff/issues/452",
        "count": 1
      }
    ],
    "days_passed_since_last_release": 0
  },
  "previous": {
    "version": "previous release"
  }
}
```

:::info

See the [GitHub integration](/docs/integration/github) for the additional values you can use in the template.

:::

## Release statistics

You can access various release-related metrics via the `statistics` variable. The following fields are available:

- `commit_count`: Total number of commits in the release.
- `commits_timespan`: Number of days between the first and last commit.
- `conventional_commit_count`: Number of commits that follow the Conventional Commits spec.
- `links`: A list of issues or links referenced in commit messages, each with text, href, and count.
- `days_passed_since_last_release`: Days since the previous release, if available.

You can use these fields in your templates like so:

```jinja2
- {{ statistics.commit_count }} commit(s) contributed to the release.
- {{ statistics.commits_timespan | default(value=0) }} day(s) passed between the first and last commit.
- {{ statistics.conventional_commit_count }} commit(s) parsed as conventional.
- {{ statistics.links | length }} linked issue(s) detected in commits.
{%- if statistics.links | length > 0 %}
	{%- for link in statistics.links %}
        {{ "  " }}- [{{ link.text }}]({{ link.href }}) (referenced {{ link.count }} time(s))
	{%- endfor %}
{%- endif %}
{%- if statistics.days_passed_since_last_release %}
	- {{ statistics.days_passed_since_last_release }} day(s) passed between releases.
{%- endif %}
```

This results in the following output:

<details>
  <summary>Rendered Output</summary>

### Commit Statistics

- 2 commit(s) contributed to the release.
- 0 day(s) passed between the first and last commit.
- 2 commit(s) parsed as conventional.
- 0 linked issue(s) detected in commits.
- 1430 day(s) passed between releases.

</details>
