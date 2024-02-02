## Purpose  
This repository is demonstrating "collaboration" between 2 Github Workflows.  

The main thing demonstrated here is how to use an artifact in one workflow, created by another.  

## Setup  
in `.github/workflows` you find 2 files:  
* `build.yml` - which is triggered by any push (any branch)
* `post_build.yml` - which is either triggered by "build.yml -> completed" (workflow_run) event - or manually (workflow_dispatch)  

For `post_build.yml` to be able to download the artifact from `build.yml`, it needs to know about the `run_id` from `build.yml`.  
This `run_id` will either be provided by the `workflow_run` event - `${{ github.event.workflow_run.id }}` -  
or by prompting the user when executing the workflow manually (workflow_dispatch):  
```yml
  workflow_dispatch:
    inputs:
      RUN_ID:
        type: text
        required: true
        description: Type in the Run ID from the Build step creating the artifact.
```

Scripts will after this be able to access the input variable by accessing `${{ inputs.RUN_ID }}` value.  

**Downloading the artifact**:  
When downloading the artifact from another workflow, the `actions/download-artifact@v4` - action, needs additional input:  
```yml
    - name: Download the Build Artifact
      uses: actions/download-artifact@v4.1.1
      with:
#        name: Artifacts_${{ env.ARTIFACTS_SHA }} # If specified, files will be extracted to the root folder!
        github-token: ${{ secrets.GITHUB_TOKEN }} # ${{ secrets.GH_PAT }}  # optional
        run-id: ${{ env.BUILD_RUN_ID }}
```
... where `${{ env.BUILD_RUN_ID }}` is being set by a setup step, either fetching the value from `${{ github.event.workflow_run.id }}` or `${{ inputs.RUN_ID }}` 
depending on which event triggered the workflow. Example:  
```
        if [[ "${{ github.event_name }}" == "workflow_dispatch" ]]
        then
           echo "Getting run id from workflow_dispatch - input: ${{ inputs.RUN_ID }}"
           echo "BUILD_RUN_ID=${{ inputs.RUN_ID }}" >> $GITHUB_ENV
        else
           echo "Getting run id from workflow_run: ${{ github.event.workflow_run.id }}"
           echo "BUILD_RUN_ID=${{ github.event.workflow_run.id }}" >> $GITHUB_ENV
        fi
```
