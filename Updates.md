# Steps to apply quarterly update

## Initial code changes
- Update release name, urls, and na_cutoff in [code/current_release.R]( https://github.com/hirscheylab/ddh/blob/master/code/current_release.R)
- Update [code/methods.md](https://github.com/hirscheylab/ddh/blob/master/code/methods.md) and supporting images
- Update `DMVER` in Makefile [code/Makefile](https://github.com/hirscheylab/ddh/blob/98e53f956439c570aefa7b8b2583cee1f84e8b2e/Makefile#L13) to match the new release name
- PR and Merge the above changes

NOTE: If the dependencies used for data generation have changed you will need to wait for the image to [finish buliding on dockerhub](https://hub.docker.com/repository/docker/dukegcb/ddh).

## Generate Data
All these steps are done on a Slurm cluster.
You will need at least 4G of storage run these steps.

Clone this repo, change into the directory and create a directory to hold slurm logs:
```
   git clone git@github.com:hirscheylab/ddh.git
   cd ddh   
   mkdir logs
```
Create a file named `config.sh` with the following contents (replacing TODO_NCBI_KEY with a valid key):
```
export DDH_BASE="."
SINGULARITY_BASE="${DDH_BASE}/singularity"
export SINGULARITY_TMPDIR="${SINGULARITY_BASE}/tmp"
export SINGULARITY_CACHEDIR="${SINGULARITY_BASE}/cache"
export SINGULARITY_IMAGEDIR="${SINGULARITY_BASE}/images"
export ENTREZ_KEY="TODO_NCBI_KEY"
```

Run build slurm job:
```
sbatch build-slurm.sh
```

After about 10 hours it should have finished.
View files in `logs` directory.
Re-run build-slurm.sh if it failed.

After successful upload data to DukeDS:
```
sbatch upload-slurm.sh
```

You can now delete the ddh directory from the slurm cluster.

## Deploy Data
To deploy the files requires access to the openshift okd console.
Make sure you are on the `depmap` project.
Click `Add to Project v` in the top right corner of the screen.
Choose `Select from Project`
Select the `download-job-template` and Click `Next`.
Click `Next` again.
Enter the name of the quarterly release matching the same case used in the filenames.
Click `Create`.
This will create a job that will run a pod to stage the data.

You can monitor the pod by looking for it in the Applications->Pods window.
Once the pod is complete re-deploy the application by:
Navigating to Applications -> Deployments -> ddh-shiny-app then click `Deploy`.


