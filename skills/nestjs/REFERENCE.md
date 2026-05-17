# NestJS — Reference

## Full AuthGuard

Reference implementation of the global `AuthGuard` with `@PublicApi()` and `@Roles()` decorators. Roles cascade: `SUPERADMIN` ⊇ `ADMIN` ⊇ `USER`.

```ts
import {
  CanActivate,
  ExecutionContext,
  ForbiddenException,
  Injectable,
  SetMetadata,
  UnauthorizedException,
} from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { UserRole } from '@auth-lib/common';
import { UserEntity, UserService } from '../user';
import { AuthRequest } from './auth.types';

enum MetadataKey {
  Roles = 'AuthGuard.Roles',
  Public = 'AuthGuard.Public',
}

export function Roles(...roles: UserRole[]): MethodDecorator & ClassDecorator {
  return SetMetadata(MetadataKey.Roles, roles);
}

export function PublicApi(): MethodDecorator & ClassDecorator {
  return SetMetadata(MetadataKey.Public, true);
}

@Injectable()
export class AuthGuard implements CanActivate {
  constructor(
    private readonly reflector: Reflector,
    private readonly userService: UserService,
  ) {}

  protected getMetadata<T>(context: ExecutionContext, key: MetadataKey): T | undefined {
    return this.reflector.getAllAndOverride<T>(key, [context.getHandler(), context.getClass()]);
  }

  protected isAllowed(user: UserEntity, roles: UserRole[]): boolean {
    const userRoles: UserRole[] = [];

    switch (user.role) {
      case UserRole.SUPERADMIN:
        userRoles.push(UserRole.SUPERADMIN);
      // fallthrough
      case UserRole.ADMIN:
        userRoles.push(UserRole.ADMIN);
      // fallthrough
      case UserRole.USER:
        userRoles.push(UserRole.USER);
    }

    return userRoles.some((role: UserRole): boolean => roles.includes(role));
  }

  async canActivate(context: ExecutionContext): Promise<boolean> {
    if (this.getMetadata<boolean>(context, MetadataKey.Public)) {
      return true;
    }

    const request = context.switchToHttp().getRequest<AuthRequest>();
    const userId: number | undefined = request.session?.userId;

    if (!userId) {
      throw new UnauthorizedException('Not authenticated');
    }

    const user: UserEntity | null = await this.userService.findById(userId);

    if (!user) {
      throw new UnauthorizedException('User not found');
    }

    if (!this.isAllowed(user, this.getMetadata<UserRole[]>(context, MetadataKey.Roles) ?? [UserRole.USER])) {
      throw new ForbiddenException('Not allowed!');
    }

    request.user = user;

    return true;
  }
}
```

## Usage

```ts
@Controller('health')
export class HealthController {
  @PublicApi()
  @Get()
  ping(): { ok: true } {
    return { ok: true };
  }
}

@Controller('admin/users')
@Roles(UserRole.ADMIN)
export class AdminUserController {
  // all routes require ADMIN or SUPERADMIN
}
```

## Module wiring

```ts
@Module({
  imports: [TypeOrmModule.forFeature([UserEntity])],
  providers: [
    UserService,
    { provide: APP_GUARD, useClass: AuthGuard },
  ],
  controllers: [UserController],
})
export class AppModule {}
```

## AuthRequest type

```ts
import { Request } from 'express';
import { UserEntity } from '../user';

export interface AuthSession {
  userId?: number;
}

export interface AuthRequest extends Request {
  session: AuthSession;
  user?: UserEntity;
}
```
