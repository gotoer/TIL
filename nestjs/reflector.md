# Reflector - 데코레이터 기능 확장자

## NestJS의 Reflector란?

NestJS에서 Reflector는 메타데이터를 읽고 활용할 수 있도록 도와주는 유틸리티 클래스입니다.  
NestJS의 데코레이터 기반 프로그래밍에서 데코레이터에 저장된 메타데이터를 읽어올 때 사용됩니다.

<br>
<br>

## Reflector 사용방법

Reflector는 보통 @nestjs/core에서 제공하는 Reflector 클래스를 사용합니다.  

```typescript
import { Reflector } from '@nestjs/core';
```

특정 라우터에 권한을 부여하는 기능을 붙이고 싶다고 가정해보도록 하겠습니다.  

<br>

1. 커스텀 데코레이터 생성  
: SetMetadata를 활용해 커스텀 데코레이터를 만들어줍니다.  

```typescript
import { SetMetadata } from '@nestjs/common';

// 'roles'라는 키로 메타데이터 저장
export const Roles = (roles: string[]) => SetMetadata('roles', roles);
```

<br>

controller에서는 역할을 지정합니다.

```typescript
@Controller('users')
export class UserController {
  @Get()
  @Roles('admin')  // 'admin' 역할을 부여
  findAll() {
    return "This route is restricted to admins.";
  }
}
```

<br>

2. Reflector로 메타데이터 읽기 (Guard에서 활용)  
: Reflector는 보통 Guard 또는 Interceptor에서 사용됩니다.
위에서 역할을 지정했으므로 RolesGuard를 만들어 특정 역할이 있는 사용자만 API를 호출할 수 있도록 할 수 있습니다.  

```typescript
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Reflector } from '@nestjs/core';

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    // @Roles() 데코레이터에서 설정한 메타데이터 가져오기
    const requiredRoles = this.reflector.get<string[]>('roles', context.getHandler());
    if (!requiredRoles) {
      return true; // 역할이 필요 없으면 접근 허용
    }

    const request = context.switchToHttp().getRequest();
    const user = request.user; // 가정: request.user에 사용자 정보가 있음

    return requiredRoles.some((role) => user.roles?.includes(role)); // 사용자 역할 검사
  }
}
```

<br>
<br>

## Reflector를 사용하는 이유?

데코레이터의 메타데이터를 쉽게 불러올 수 있고, 그로 인해 가드나 인터셉터에서 동적으로 정책 적용이 가능하기 때문입니다.
