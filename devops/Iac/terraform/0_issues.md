# Issues

## How to change the name of workspace

1. `terraform workspace select old-name`
2. `terraform state pull > <old-name>.tfstate`
3. `terraform workspace new <new-name>`
4. `terraform state push <old-name>.tfstate`
5. `terraform show` just to confirm that the newly-imported state looks “right”, before we delete the old workspace 
6. `terraform workspace delete -force <old-name>`

## Error: Failed to install provider

M1 Mac은 darwin64 os를 사용한다. 그래서 간혹 terraform에서 지원하는 기능을 못 쓸 때가 있는데, 이를 해결해주는 방법을 알려주겠다.

1. 아래 명령어 실행

   ```sh
   brew install kreuzwerker/taps/m1-terraform-provider-helper
   m1-terraform-provider-helper activate
   ```

2. `.terraform.lock.hcl`파일과 `.terraform` 폴더 삭제

3. 아래 명령어 실행

   ```sh
   m1-terraform-provider-helper install hashicorp/template -v v2.2.0
   tf init
   tf providers lock -platform=darwin_amd64
   tf apply
   ```

## Invalid legacy provider address

```sh
tf state replace-provider -- -/aws hashicorp/aws
```