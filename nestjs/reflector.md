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

한 곳에서만 사용된다면 Roles라는 데코레이터명을 쓰지 않고 `@SetMetadata('roles', roles)` 이렇게 사용해도 됩니다.

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

## 주요 사용하는 메서드

nestjs reflector에서 주로 사용할만한 함수는 아래와 같습니다.

- get<T>(metadataKey, target) : @Roles() 데코레이터로 설정된 단일 핸들러의 메타데이터만 가져옵니다.  

```typescript
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.get<string[]>('roles', context.getHandler());
    console.log('Required Roles:', requiredRoles); // ['admin', 'editor']
    return true;
  }
}
```

context.getHandler()는 현재 실행 중인 컨트롤러의 특정 메서드를 의미하고, reflector.get('roles', context.getHandler())을 통해 해당 핸들러에 설정된 역할 목록을 가져옵니다.

<br>

- getAll<T>(metadataKey, targets[]) : 클래스와 메서드에 있는 모든 roles 값을 가져옵니다.

```typescript
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    // 클래스와 메서드의 메타데이터를 각각 가져옴
    const roles = this.reflector.getAll<string[]>('roles', [
      context.getHandler(), // 현재 실행 중인 메서드
      context.getClass(),   // 현재 실행 중인 컨트롤러 클래스
    ]);

    console.log('Roles from method:', roles[0]); // ['admin']
    console.log('Roles from class:', roles[1]);  // ['user']

    return true;
  }
}
```

context.getClass()는 현재 실행 중인 클래스(컨트롤러)를 의미합니다.

<br>

- getAllAndOverride<T>(metadataKey, targets[]) : 클래스와 메서드에서 가져오되, 가장 최근(우선순위가 높은) 값을 반환합니다.

```typescript
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    // 메서드에 'roles'가 있으면 가져오고, 없으면 클래스의 'roles'를 가져옴
    const requiredRoles = this.reflector.getAllAndOverride<string[]>('roles', [
      context.getHandler(),
      context.getClass(),
    ]);

    console.log('Required Roles:', requiredRoles); // ['admin'] (메서드에 설정된 값이 우선)
    return true;
  }
}
```

<br>

- getAllAndMerge<T>(metadataKey, targets[]) : @Roles() 데코레이터가 클래스와 메서드에 동시에 설정되어 있을 경우, 이를 합쳐서 반환합니다.

```typescript
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    // 클래스와 메서드의 'roles'를 합쳐서 가져옴
    const requiredRoles = this.reflector.getAllAndMerge<string[]>('roles', [
      context.getHandler(),
      context.getClass(),
    ]);

    console.log('Merged Roles:', requiredRoles); // ['admin', 'user']
    return true;
  }
}
```

<br>
<br>

## Reflector를 사용하는 이유?

데코레이터의 메타데이터를 쉽게 불러올 수 있고, 그로 인해 가드나 인터셉터에서 동적으로 정책 적용이 가능하기 때문입니다.  
권한 관리 밖에도 로깅, 캐싱 등 다양한 곳에서도 활용할 수 있습니다.
