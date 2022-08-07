## Yarn2 적용

이제 본격적으로 환경을 구축해보겠습니다. 물론 위에서 **PnP** 방식을 설명하며 강조하였지만, 여전히 `node_modules` 방식을 사용할 수 있습니다. 이에 대해서는 아래에서 다루겠습니다.

> 1. 먼저 **Yarn**을 설치
>
> ```shell
> npm install -g yarn
> ```

> 2. 프로젝트의 경로로 이동하여, 다음과 같이 `berry` 버전으로 설정
>
> ```shell
> yarn set version berry
> yarn -v
> > 3.x.x
> ```
>
> ***+ 2021.08.21\***
> 아래는 위 방법이 안 되시는 경우가 발생하여, 추가적으로 작성합니다!
>
> ```shell
> yarn policies set-version
> ```

> 1. 기존 관련 파일을 수정해보도록 하겠습니다.
>
> - `.npmrc` 파일을 `.yarnrc.yml` 으로 수정 [[관련 옵션\]](https://yarnpkg.com/configuration/yarnrc)
> - `package.lock.json` 파일이 존재한다면 제거
> - `node_modules` 폴더 제거
> - `package.json` 파일에 작성된 `eslintConfig`는 더 이상 사용되지 않습니다. `.eslintrc.json` 파일로 빼주세요.

> 2. **[선택사항]** `PnP` 방식을 사용하지 않겠다면, `.yarnrc.yml` 파일에 다음 내용을 추가해주세요.
>
> ```shell
> nodeLinker: node-modules
> ```
>
> 해당 옵션을 작성하면 `zip 아카이브`로 관리되는 것이 아닌 기존의 `node_modules` 방식으로 관리되게 됩니다.

> 3. 설치를 진행해줍니다. 설치를 하고 **4번 과정**을 진행하지 않았다면, `.yarn`폴더가 생긴 것을 확인할 수 있습니다.
>
> ```shell
> yarn install 
> # 또는
> yarn
> ```

> 4. `.gitignore` 파일에 다음 내용을 중 하나를 추가해주세요.
>
> ```shell
> ### yarn ###
> # Zero-Install을 사용하겠다면?
> .yarn/*
> !.yarn/cache
> !.yarn/patches
> !.yarn/plugins
> !.yarn/releases
> !.yarn/sdks
> !.yarn/versions
> 
> # Zero-Install을 사용하지 않겠다면?
> .yarn/*
> !.yarn/patches
> !.yarn/releases
> !.yarn/plugins
> !.yarn/sdks
> !.yarn/versions
> .pnp.*
> ```

이제 정상적으로 개발을 진행할 수 있습니다!

아래 과정은 만약 타입스크립트를 추가 적용하고 싶은 분들을 대상으로 진행하는 내용입니다. 이제 `@types/*` 패키지들이 zip 아카이브로 관리되기 때문에 기존의 방식으로는 정상적으로 타입이 불러와지지 않을 것입니다. 그렇기에 다음과 같은 과정을 진행해주세요!

설명은 **VSCode** IDE 기준으로 설명되었습니다.

> 1. 타입스크립트가 적용되지 않은 프로젝트라면 타입스크립트 관련 패키지를 설치해주세요.
>    **[ 아래는 코드 React를 사용한다는 기준으로 작성된 스크립트입니다. ]**
>
> ```shell
> yarn add -D typescript @types/node @types/react @types/react-dom @types/jest
> ```

> 2. `jsconfig.json`대신 `tsconfig.json`으로 작성해주세요.
>
> 3. `ZipFS` 확장자를 VSCode에 검색해서 설치해주세요.
>
> 4. Typescript 적용
>
>    ```shell
>    yarn plugin import workspace-tools
>    ```
>
>    프로젝트에서 Typescript를 사용하는 경우 Yarn berry와 호환될 수 있도록 위 플러그인을 설치한다
>
> 5. 해당 프로젝트 최상위 경로에서 다음을 입력합니다.
>    (기존 방식이 deprecated되어, 새로운 방식으로 수정하였습니다.)
>    
>    ```shell
>    yarn dlx @yarnpkg/sdks vscode
>    ```
>    
> 5. VSCode를 실행하여 Typescript 버전을 선택해주세요.
>    ![img](https://media.vlpt.us/images/altmshfkgudtjr/post/0f560e77-332d-4502-af32-2b9c220f673a/image.png)위 사진과 같이 `version-pnpify` 버전을 선택해야 합니다. **[작업 영역 버전 사용]**

이제 정상적으로 타입이 호출되며, 개발을 진행할 수 있습니다!

### 몇 가지 유의사항

1. **Node**를 실행시킬 때, 기존의 `node`로 스크립트를 시작하는 것이 아닌, `yarn node`로 스크립트를 실행시켜야 합니다.
2. 만약 `styled-components` 패키지를 사용하고 계시다면 추가적으로 `react-is` 패키지를 설치해야만 정상적으로 작동하는 이슈가 있습니다.kaka