# Commands

1. feat/ 이하의 모든 브렌치 한 번에 지우기

   ```sh
   git branch | grep "feat/" | xargs git branch -D
   ```