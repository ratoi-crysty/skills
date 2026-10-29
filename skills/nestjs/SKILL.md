---
name: nestjs
description: Conventions for authoring NestJS backends — TypeORM entities inheriting shared model interfaces, CRUD-first controllers, Swagger/OpenAPI documentation, and a global AuthGuard with explicit opt-out via `@PublicApi()`. Use when building or refactoring NestJS modules, controllers, services, entities, guards, or DTOs, or when the user mentions NestJS, TypeORM, `@nestjs/swagger`, NX backend libs, `APP_GUARD`, or CRUD endpoints.
---

# NestJS

## Core rules

1. **TypeORM** for persistence — entities, repositories, migrations.
2. **Shared model interface** in an NX `common` lib; `Entity` classes implement it so DTOs/clients share the contract.
3. **CRUD-first** — drop to bespoke handlers only when CRUD cannot express the operation.
4. **Swagger everywhere** — every controller, route, DTO, and entity decorated with `@nestjs/swagger` so OpenAPI is complete.
5. **Global AuthGuard** registered via `APP_GUARD`; routes are protected by default and opt out with `@PublicApi()`.
6. **Folder structure** — The folder structure should be inside app (`src/app/`) if the main module (`app.module.ts`) is inside app folder as well. This should be relevant mainly for NX monorepos.

## Shared model + entity (NX)

```ts
// libs/user/common/src/lib/user.model.ts
export interface UserModel {
  id: number;
  email: string;
  role: UserRole;
}
```

```ts
// apps/api/src/app/user/user.entity.ts
import { UserModel, UserRole } from '@user-lib/common';
import { ApiProperty } from '@nestjs/swagger';
import { Column, Entity, PrimaryGeneratedColumn } from 'typeorm';

@Entity('users')
export class UserEntity implements UserModel {
  @ApiProperty()
  @PrimaryGeneratedColumn()
  id: number;

  @ApiProperty()
  @Column({ unique: true })
  email: string;

  @ApiProperty({ enum: UserRole })
  @Column({ type: 'enum', enum: UserRole })
  role: UserRole;
}
```

- One interface per aggregate in the `common` lib. Reused by frontend, DTOs, and the entity.
- Entity adds persistence/Swagger decorators; never duplicate field types.

## CRUD controller

```ts
@Crud({
  model: { type: UserEntity },
  dto: { create: CreateUserDto, update: UpdateUserDto },
})
@ApiTags('users')
@Controller('users')
export class UserController implements CrudController<UserEntity> {
  constructor(public service: UserService) {}
}
```

- Custom routes only when CRUD cannot express the operation. Document each with `@ApiOperation`, `@ApiResponse`.
- DTOs use `@ApiProperty` + `class-validator` decorators.

## Global AuthGuard

Register once:

```ts
@Module({
  providers: [{ provide: APP_GUARD, useClass: AuthGuard }],
})
export class AppModule {}
```

Guard implements:

- `@PublicApi()` — bypass auth for a route or controller.
- `@Roles(...roles)` — restrict to specific roles; default required role is `UserRole.USER`.
- Resolves session → loads `UserEntity` → attaches to `request.user`.

See [REFERENCE.md](REFERENCE.md) for the full guard implementation.

## Module layout

```
apps/api/src/app/user/
├── user.module.ts
├── user.controller.ts        # @Crud controller
├── user.service.ts           # extends TypeOrmCrudService<UserEntity>
├── user.entity.ts            # implements UserModel
├── dto/
│   ├── create-user.dto.ts
│   └── update-user.dto.ts
└── user.types.ts             # local-only types
```

- Shared types/interfaces live in the NX `common` lib, not the app.
- DTOs stay app-side — they describe transport, not the domain.

## Swagger bootstrap

```ts
const config = new DocumentBuilder()
  .setTitle('API')
  .setVersion('1.0')
  .addCookieAuth('session')
  .build();
SwaggerModule.setup('docs', app, SwaggerModule.createDocument(app, config));
```

## Checklist before committing

- [ ] Entity implements a `common`-lib model interface
- [ ] Controller is `@Crud`-based unless justified otherwise
- [ ] Every route, DTO, and entity field has Swagger decorators
- [ ] `AuthGuard` registered via `APP_GUARD` in the root module
- [ ] Public routes explicitly tagged with `@PublicApi()`
- [ ] Role-restricted routes tagged with `@Roles(...)`
- [ ] No `any` in controller/service signatures (see [typescript-standards](../typescript-standards/SKILL.md))
