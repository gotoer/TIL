# Typeorm EntityManager 함수 increment와 update는 어떤 차이가 있을까?

## 코드 사용 예시

update, increment 함수를 사용한 코드는 아래와 같습니다.  

``` typescript
// update 사용 시
await manager.update(
  Question, 
  { postId }, 
  { answerCount: ++question.answerCount }  // 메모리에서 값을 증가시키고 업데이트
);
```

``` typescript
// increment 사용 시
await manager.increment(
  Question,
  { postId },
  'answerCount',
  1
);
```

<br>
<br>

## 주요 차이점

카운터를 증가시키는 경우에는 increment를 사용하는 것이 더 안전하고 효율적입니다.  
무엇보다 동시성 처리면에서 차이가 있고 성능에서도 차이가 납니다.  

1. 동시성 처리:

- update
  - 메모리에서 값을 증가시키고 업데이트하므로 race condition이 발생할 수 있습니다
- increment
  - DB 레벨에서 atomic하게 증가하므로 race condition 걱정이 없습니다  

<br>

2. 성능:  

- update
  - 현재 값을 먼저 읽어와야 하므로 추가 쿼리가 필요할 수 있습니다
- increment
  - 단일 쿼리로 처리되므로 더 효율적입니다

<br>
<br>

## 차이가 나는 이유?

우선 쿼리에서도 차이가 납니다. typeorm update는 단순히 SQL의 UPDATE 문을 생성해 엔티티의 특정 필드를 직접 지정한 값으로 설정합니다.

``` sql
-- 첫 번째 쿼리 (조회)
SELECT * FROM "entity" WHERE "id" = 1;

-- 두 번째 쿼리 (업데이트)
UPDATE "entity" SET "count" = 6 WHERE "id" = 1;
```

<br>

typeorm increment는 TypeORM에서 특별히 카운터 증가를 위해 최적화된 메서드입니다. 데이터베이스 내에서 현재 값을 기준으로 증가시키는 SQL을 생성합니다.

``` sql
-- 첫 번째 쿼리 (업데이트)
UPDATE "entity" SET "count" = "count" + 1 WHERE "id" = 1;
```

<br>

쿼리 요청을 실행하는 postgres 측면에서도 update는 일반 update를 increment는 증분 update를 실행합니다.  
그 차이는 아래와 같습니다.  

- 일반 UPDATE: 값을 직접 설정하는 방식은 여러 트랜잭션이 동시에 실행될 때 마지막 트랜잭션의 값으로 덮어쓰게 됩니다.  
- 증분 UPDATE: count = count + 1과 같은 방식은 PostgreSQL이 트랜잭션 격리 수준에 따라 적절히 처리하여 동시성 문제를 해결합니다.  

<br>
<br>

## 결론

``` typescript
// increment 구현체
increment<Entity>(
    entityClass: EntityTarget<Entity>,
    conditions: any,
    propertyPath: string,
    value: number | string,
): Promise<UpdateResult> {
    // 메타데이터 가져오기
    const metadata = this.connection.getMetadata(entityClass)
    
    // 쿼리 빌더 생성
    const qb = this.createQueryBuilder<Entity>(entityClass, "entity")
    
    // 조건 적용
    Object.keys(conditions).forEach((key) => {
        const value = conditions[key]
        if (value === undefined || value === null) return

        qb.andWhere(`entity.${key} = :${key}`, { [key]: value })
    })
    
    // 증분 쿼리 생성 및 실행
    return qb
        .update()
        .set({
            [propertyPath]: () => `${propertyPath} + ${value}`,
        } as any)
        .execute()
}
```

TypeORM의 increment 메서드는 특별한 동시성 처리 코드를 포함하지 않지만, 생성하는 SQL 쿼리의 특성과 데이터베이스의 트랜잭션 관리 메커니즘을 활용하여 동시성 문제를 해결합니다.  

이것이 increment 메서드가 카운터 증가와 같은 작업에 더 안전한 이유입니다.
