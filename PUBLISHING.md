# Publish to the Gas City registry

The current release is `tavily-fde/tavily` version `0.1.1`. The repository owner
is `Tavily-FDE`; Gas City requires the pack scope to match its lowercase form.
Keep the scoped name in `pack.toml` identical to the submitted name.

## Validate before submitting

1. Confirm `gc version` reports 1.5.0 or later.
2. Import the intended commit into a test city with the README's import command
   and `--version 'sha:<full-commit-sha>'`.
3. Run `gc import check`, `gc config show --validate`, and
   `gc mcp list --agent <agent-name>`. Confirm the Tavily HTTP server appears.
4. Authenticate in the consuming client and run one Search and one Extract
   using the README's prompts. A config check alone does not test OAuth or
   live tools.
5. Commit and push all release files. For the browser flow, merge them into
   the repository's default branch before scanning.
6. Tag the release commit `v<version>` (for example `v0.1.1`) and push the tag,
   so `gc import add --version '^0.1.1'` can resolve it.

## Submit through GitHub App discovery

1. Install the [Gas City Registry Verifier](https://github.com/apps/gas-city-registry-verifier/installations/select_target)
   under **Tavily-FDE**, selecting **Only select repositories** and
   **tavily-gascity-pack**. Use a GitHub account with repository admin access;
   an organization owner may need to approve the installation.
2. Review the actual GitHub consent screen. At preparation time the app
   requested **Contents: read** and **Metadata: read**. No repository write
   permission is needed for this flow.
3. Sign in at <https://registry.gascity.com/publish> and choose **Find packs**.
   Complete GitHub verification and select this repository's root pack.
4. Review the discovered commit and these values:

   | Field | Value |
   |---|---|
   | Repository | `Tavily-FDE/tavily-gascity-pack` |
   | Pack path | `.` |
   | Pack name | `tavily-fde/tavily` |
   | Version | `0.1.1` |
   | Ref label | The default branch, normally `main` |

5. Click **Submit this pack** and follow the request on the Account page.
   Fix any validation errors before resubmitting. **Pending review** means
   queued; only **Approved** means the release is served in the public catalog
   and CLI registry.

Discovery submits the default branch's current HEAD. Changing the ref label
does not select a different commit. Versions must be stable `major.minor.patch`
strings such as `0.1.0`, without a leading `v` or prerelease suffix.

The registry computes the content hash from the submitted public source;
this direct flow does not require a checked-in `registry.toml` or a change to
Gas City's aggregate repository. Future releases still require staff review.
For tagged releases, follow the official
[GitHub Actions OIDC publishing instructions](https://registry.gascity.com/publish).
Creating or merging a GitHub PR does not itself submit a registry release.
