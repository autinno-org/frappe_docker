

### Release Wizard
<!-- https://github.com/frappe/helm/tree/main/release_wizard -->

This is a release script for maintainers. It does the following:

- Checks latest tag for given major release for frappe and erpnext using git.
- Validates that release always bumps up.
- Bumps values.yaml and Chart.yaml for release changes
- Adds git tag for chart version
- Push to selected remote

This will trigger workflow to publish new version of helm chart.

- .github/workflows/build_stable.yml

  release_helm:
    name: Release Helm
    runs-on: ubuntu-latest
    if: ${{ github.repository == 'frappe/frappe_docker' && github.event_name != 'pull_request' }}
    needs: v15

    steps:
      - name: Setup deploy key
        uses: webfactory/ssh-agent@v0.9.1
        with:
          ssh-private-key: ${{ secrets.HELM_DEPLOY_KEY }}

      - name: Setup Git Credentials
        run: |
          git config --global user.email "41898282+github-actions[bot]@users.noreply.github.com"
          git config --global user.name "github-actions[bot]"

      - name: Release
        run: |
          git clone git@github.com:frappe/helm.git && cd helm
          pip install -r release_wizard/requirements.txt
          ./release_wizard/wizard 16 patch --remote origin --ci

- https://github.com/frappe/erpnext/blob/develop/.github/workflows/docker-release.yml

  v16:
    uses: ./.github/workflows/docker-build-push.yml
    with:
      repo: erpnext
      version: "16"
      push: ${{ github.repository == 'frappe/frappe_docker' && github.event_name != 'pull_request' }}
      python_version: 3.14.2
      node_version: 24.12.0
    secrets:
      DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}
