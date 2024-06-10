# Issues

## How to change the name of workspace

1. `terraform workspace select old-name`
2. `terraform state pull > <old-name>.tfstate`
3. `terraform workspace new <new-name>`
4. `terraform state push <old-name>.tfstate`
5. `terraform show` just to confirm that the newly-imported state looks “right”, before we delete the old workspace 
6. `terraform workspace delete -force <old-name>`

## Error: Failed to install provider

MAC M1 : `terraform providers lock -platform=darwin_amd64`

## Invalid legacy provider address

```sh
tf state replace-provider -- -/aws hashicorp/aws
```