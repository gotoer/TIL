# Typeorm-merge

typeorm의 repository를 자주 활용하고 있고 주로 create, insert, save, update를 사용하다가 merge를 처음보게 되었습니다.  
preload는 간간히 본적이 있었지만 자주 사용하지 않기에 넘어갔었는데 merge가 나오게 되면서 기능 정리를 하고자 합니다.

## Repository?

TypeORM에서 Repository는 특정 엔터티에 대한 데이터베이스 작업을 수행하는 객체입니다.  
데이터를 조회, 생성, 수정, 삭제(CRUD)할 수 있는 도구라고 볼 수 있습니다.  

<br>

특정 엔티티의 repository를 가져오는 코드는 아래와 같습니다.

``` typescript
import { AppDataSource } from "./data-source";
import { User } from "./entities/User";

const userRepository = AppDataSource.getRepository(User);
```

<br>
<br>

## 엔터티 객체를 생성하는 메서드

Typescript를 사용하다보니 특정 엔티티 타입을 보장하기 위해 typeorm의 엔티티 객체를 사용하는 메서드를 사용하게 됩니다.
엔티티 객체를 생성하는 메서드는 create(), preload(), merge()가 있습니다.  

각각의 특징은 아래와 같습니다.

1. repository.create()
    - 새로운 엔터티 객체를 생성하지만, 데이터베이스에 저장하지는 않습니다.
    - 전달된 데이터로 엔터티 인스턴스를 생성하지만, id 같은 기본키는 자동 할당되지 않습니다.
    - 데이터베이스에서 기존 엔터티를 찾지 않습니다.
    - save()를 호출해야 데이터베이스에 저장됩니다.

[사용 예시]

``` typescript
const user = userRepository.create({ name: "John", age: 25 });

await userRepository.save(user);
```

<br>

2. repository.merge()
    - 기존 엔터티 인스턴스에 새로운 데이터를 병합합니다.
    - 주어진 데이터로 기존 엔터티의 필드를 업데이트하지만, 새로운 인스턴스를 생성하지 않습니다.
        - 특정 객체의 메모리 주소(레퍼런스)는 그대로 유지되지만, 필드 값이 변경됨을 의미합니다.
    - 기존 엔터티 객체가 필요합니다. 즉, findOne() 등으로 먼저 가져온 후 사용해야 합니다.

[사용 예시]

``` typescript
const user = await userRepository.findOne({ where: { id: 1 } });

if (user) {
  userRepository.merge(user, { age: 30 }); // 인스턴스를 생성하지 않음
  await userRepository.save(user);
}
```

3. repository.preload()
    - 주어진 데이터로 엔터티를 조회하고, 존재하면 업데이트, 존재하지 않으면 null 반환합니다.
    - 기존 엔터티를 찾아서 데이터를 병합하지만, merge()와 달리 findOne()을 따로 호출할 필요가 없습니다.
    - 데이터베이스에서 먼저 엔터티를 조회 후, 병합하므로, 반드시 id(기본키)가 포함되어 있어야 합니다.
    - create()처럼 save()를 호출해야 실제로 DB에 저장됩니다.

[사용 예시]

``` typescript
const user = await userRepository.preload({ id: 1, age: 30 });

if (user) {
  await userRepository.save(user);
} else {
  console.log("User not found");
}
```

<br>
<br>

## 실제 사용 시나리오

각각의 메서드를 어떨 때 활용하면 좋을지 아래와 같이 정리해보았습니다.

- Create() : 새 데이터를 만드는 경우
  - 입력받은 데이터만으로 새로운 엔터티를 만들고 싶을 때
  - 데이터베이스에서 기존 엔터티를 조회할 필요가 없을 때

<br>

- Merge(): 이미 조회한 엔터티를 업데이트하는 경우
  - 이미 조회한 엔터티 객체가 있고, 해당 객체를 업데이트할 때
  - findOne()을 먼저 실행해서 엔터티를 가져온 후, 변경할 때
  - 여러 데이터를 한 번에 업데이트할 때 (Object.assign() 느낌으로 사용)

<br>

- Preload(): 존재하면 업데이트, 없으면 무시해야하는 경우
  - merge()처럼 엔터티를 업데이트하지만, findOne()을 직접 호출할 필요가 없을 때
  - 해당 ID의 데이터가 없으면 굳이 새로 만들 필요 없을 때
  - 자동으로 조회 후 병합하는 기능이 필요할 때
