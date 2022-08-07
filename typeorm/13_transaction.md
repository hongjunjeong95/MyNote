# Transaction

**Ref**

* [typeorm transcation](https://typeorm.io/#/transactions)

## users.service.ts

```typescript
  async join(email: string, nickname: string, password: string) {
    const queryRunner = this.connection.createQueryRunner();
    await queryRunner.connect();
    await queryRunner.startTransaction();
    const user = await queryRunner.manager
      .getRepository(Users)
      .findOne({ where: { email } });
    if (user) {
      throw new ForbiddenException('이미 존재하는 사용자입니다');
    }
    const hashedPassword = await bcrypt.hash(password, 12);
    try {
      const returned = await queryRunner.manager.getRepository(Users).save({
        email,
        nickname,
        password: hashedPassword,
      });
      const workspaceMember = queryRunner.manager
        .getRepository(WorkspaceMembers)
        .create();
      workspaceMember.UserId = returned.id;
      workspaceMember.WorkspaceId = 1;
      await queryRunner.manager
        .getRepository(WorkspaceMembers)
        .save(workspaceMember);
      await queryRunner.manager.getRepository(ChannelMembers).save({
        UserId: returned.id,
        ChannelId: 1,
      });
      await queryRunner.commitTransaction();
      return true;
    } catch (error) {
      console.error(error);
      await queryRunner.rollbackTransaction();
      throw error;
    } finally {
      await queryRunner.release();
    }
  }
```

* @Transaction 을 쓰면 서비스 DI가 힘들어진다. 테스트할 때도 DI가 어려워진다.
* 성공하면 queryRunner.commitTranscation()을 꼭 해야 한다.
* 실패했을 때는 queryRunner.rollbackTransaction()을 해줘야 한다. `queryRunner.startTransaction()` 상태로 돌아간다.
* 성공했든 실패했든 마지막에는 `queryRunner.release()`를 해줘서 연결을 끊어줘야 한다. RDB는 최대 연결 개수가 있는데 연결이 쌓이면서 새로운 연결을 하지 못 하게 되어 쿼리가 실행되지 못 한다.
