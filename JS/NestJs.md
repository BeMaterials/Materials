- Install it by using `npm i -g @nestjs/cli` and verify it by `nest --version`.

# App Structure

![](/md/189.jpg)

![](/md/190.jpg)

![](/md/191.jpg)

# Installation

- Create a new project by `nest new my-project`.

![](/md/175.jpg)

- `tslint.json` is for typescript linting configuration.
- `tsconfig.json` and `tsconfig.build.json` is for typescript compiler configuration in development and also specific when it is built.
- `src/main.ts` is the entry point for our app:

```js
import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module";

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();
```

- `NestJs` uses `express` under the hood.
- The `AppModule` is the root module.
- `src/app.module.ts`:

```js
import { Module } from "@nestjs/common";

@Module({
  imports: [],
})
export class AppModule {}
```

- Clean other files and then `npm run start:dev`.

# Modules

- Treat your modules in isolated way as you can.

![](/md/192.jpg)

- A module is defined by annotating a class with `@Module` decorator.
- The decorator provides metadata that Nest uses to organize the app structure.

![](/md/193.jpg)

![](/md/194.jpg)

![](/md/195.jpg)

- To generate a module: `nest g module tasks`. The `tasks` is the path relative to `src` folder.
- Nest creates a folder inside `src` with the module name, creates the module file (`tasks.module.ts`), and changes the root module to import the newly-created module.

```js
import { Module } from "@nestjs/common";

@Module({})
export class TasksModule {}
```

```js
import { Module } from "@nestjs/common";
import { TasksModule } from "./tasks/tasks.module";

@Module({
  imports: [TasksModule],
})
export class AppModule {}
```

- So Nest/CLI will write the boilerplate code for us.

# Controllers

![](/md/196.jpg)

- A controller is defined by decorating a class with `@Controller` decorator.
- The decorator accepts a string which is the path to be handled by the controller.
- Handlers are defined inside the contoroller.

![](/md/197.jpg)

![](/md/198.jpg)

- To generate a controller to route all incoming requests from `/tasks` to it: `nest g controller tasks --no-spec`. (`--no-spec` because we don't write any unit tests right now).
- Nest creates `tasks.controller.ts` inside `tasks` sub-folder and modifies `tasks` module to import that controller.

```js
import { Controller } from "@nestjs/common";

@Controller("/tasks")
export class TasksController {}
```

```js
import { Module } from "@nestjs/common";
import { TasksController } from "./tasks.controller";

@Module({
  controllers: [TasksController],
})
export class TasksModule {}
```

- Remember that `decorators` annotates classes or their members (methods or properties) to add extra functionality.

# Providers and Services

![](/md/199.jpg)

![](/md/200.jpg)

![](/md/201.jpg)

![](/md/202.jpg)

- We will create a `Tasks Service` to handle all the business logic related to tasks. It is like repository class (actually it is one level of abstraction above repository). It is not concerned with routing requests (separation of concerns). Of course `Tasks Controller` requires tasks service.
- To generate a service: `nest g service tasks --no-spec`.
- Nest creates `tasks.service.ts` inside `tasks` sub-folder and modifies `tasks` module to provide that service.

```js
import { Injectable } from "@nestjs/common";

@Injectable() // It can be injected in components of the SAME module
export class TasksService {}
```

```js
import { Module } from "@nestjs/common";
import { TasksController } from "./tasks.controller";
import { TasksService } from "./tasks.service";

@Module({
  controllers: [TasksController],
  providers: [TasksService],
})
export class TasksModule {}
```

# Dependency Injection in Nest

![](/md/203.jpg)

- We can inject the service in the controller:

```ts
import { Controller } from "@nestjs/common";
import { TasksService } from "./tasks.service";

@Controller("/tasks")
export class TasksController {
  constructor(private tasksService: TasksService) {}
}
```

# DTOs

![](/md/204.jpg)

![](/md/205.jpg)

- Classes are the way to go for DTOs. Because we need them for validation in run-time.

![](/md/206.jpg)

# PATCH Best Practices

![](/md/207.jpg)

# RESTful API

- `Tasks Model`, we need to define the shape of the tasks, it can be interface or class (interface will be gone after `TS` compilation but class will be preserved as `JS` has class concept in it). It is better to start as an interface and upgrade it to class if needed. So `task.model.ts` (NOTE: we will remove when we have TypeORM entity):

```ts
export interface Task {
  id: string;
  title: string;
  description: string;
  status: TaskStatus;
}

export enum TaskStatus {
  OPEN = "OPEN",
  IN_PROGRESS = "IN_PROGRESS",
  DONE = "DONE",
}
```

- Create a folder `src/tasks/dto` with `create-task.dto.ts` file in it:

```ts
export class CreateTaskDto {
  title: string;
  description: string;
}
```

```ts
export class GetTasksFilterDto {
  status: TaskStatus;
  search: string;
}
```

- `Tasks Service`:

```ts
import { Injectable } from "@nestjs/common";
import { v1 as uuid } from "uuid";
import { Task, TaskStatus } from "./task.model";
import { CreateTaskDto } from "./dto/create-task.dto";
import { GetTasksFilterDto } from "./dto/get-tasks-filter.dto";

@Injectable()
export class TasksService {
  private tasks: Task[] = [];

  getAllTasks(): Task[] {
    return this.tasks;
  }

  getTasksWithFilters(filterDto: GetTasksFilterDto): Task[] {
    const { status, search } = filterDto;

    let tasks = this.getAllTasks();

    if (status) {
      tasks = tasks.filter((task) => task.status === status);
    }

    if (search) {
      tasks = tasks.filter(
        (task) =>
          task.title.includes(search) || task.description.includes(search)
      );
    }

    return tasks;
  }

  getTaskById(id: string): Task {
    return this.tasks.find((task) => task.id === id);
  }

  createTask(createTaskDto: CreateTaskDto): Task {
    const { title, description } = createTaskDto;

    const task: Task = {
      id: uuid(),
      title,
      description,
      status: TaskStatus.OPEN,
    };

    this.tasks.push(task);
    return task;
  }

  deleteTask(id: string): void {
    this.tasks = this.tasks.filter((task) => task.id !== id);
  }

  updateTaskStatus(id: string, status: TaskStatus): Task {
    const task = this.getTaskById(id);

    task.status = status;

    return task;
  }
}
```

- `Tasks Controller`:

```ts
import {
  Controller,
  Get,
  Post,
  Body,
  Param,
  Delete,
  Patch,
  Query,
} from "@nestjs/common";
import { TasksService } from "./tasks.service";
import { Task, TaskStatus } from "./task.model";
import { CreateTaskDto } from "./dto/create-task.dto";
import { GetTasksFilterDto } from "./dto/get-tasks-filter.dto";

@Controller("/tasks")
export class TasksController {
  constructor(private tasksService: TasksService) {}

  @Get()
  getTasks(@Query() filterDto: GetTasksFilterDto): Task[] {
    if (Object.keys(filterDto).length) {
      return this.tasksService.getTasksWithFilter(filterDto);
    } else {
      // it is very common that name of controller and service method be the same but not here
      return this.tasksService.getAllTasks();
    }
  }

  @Get("/:id")
  getTaskById(@Param("id") id: string): Task {
    return this.tasksService.getTaskById(id);
  }

  @Post()
  createTask(
    // @Body("title") title: string, // you could have access to entire body by `@Body() body`
    // @Body("description") description: string
    @Body() createTaskDto: CreateTaskDto
  ): Task {
    return this.tasksService.createTask(createTaskDto);
  }

  @Delete("/:id")
  deleteTask(@Param("id") id: string): void {
    // Nest already knows that if we do not return anything and there is no error, the response should be 200 OK
    this.tasksService.deleteTask(id);
  }

  @Patch("/:id/status")
  updateTaskStatus(
    @Param("id") id: string,
    @Body("status") status: TaskStatus
  ): Task {
    return this.tasksService.updateTaskStatus(id, status);
  }
}
```

# Validation and Error Handling

## Pipes

![](/md/208.jpg)

![](/md/209.jpg)

![](/md/210.jpg)

### Handler-level Pipes (Like middlewares)

![](/md/211.jpg)

![](/md/215.jpg)

### Parameter-level Pipes

![](/md/212.jpg)

### Global Pipes

![](/md/213.jpg)

![](/md/214.jpg)

## Add Validation

- Run `npm i class-validator class-transformer`.
- Add this to DTO class:

```ts
import { IsNotEmpty } from "class-validator";

export class CreateTaskDto {
  @IsNotEmpty()
  title: string;

  @IsNotEmpty()
  description: string;
}
```

```ts
import { IsOptional, IsIn, IsNotEmpty } from "class-validator";
import { TaskStatus } from "../task-status.enum";

export class GetTasksFilterDto {
  @IsOptional()
  @IsIn([TaskStatus.OPEN, TaskStatus.IN_PROGRESS, TaskStatus.DONE])
  status: TaskStatus;

  @IsOptional()
  @IsNotEmpty()
  search: string;
}
```

- Then in controller:

```ts
@Get()
getTasks(
  @Query(ValidationPipe) filterDto: GetTasksFilterDto, // here it is parameter-level pipe so we add it to query
): Task[] {
  if (Object.keys(filterDto).length) {
      return this.tasksService.getTasksWithFilter(filterDto);
    } else {
      return this.tasksService.getAllTasks();
    }
}

//...

@Post()
@UsePipes(ValidationPipe) // here it is handler-level pipe. Both types are coming from "@nestjs/common
createTask(
  @Body() createTaskDto: CreateTaskDto
): Task {
  return this.tasksService.createTask(createTaskDto);
}
```

![](/md/216.jpg)

## Add Error Handling

- Change service method to:

```ts
getTaskById(id: string): Task {
    const taskInDb = this.tasks.find((task) => task.id === id);

    if (!taskInDb) {
      throw new NotFoundException(); // defined in '@nestjs/common'
      // throw new NotFoundException(`Task with ID ${id} not found`); // You can use custom message
    }

    return taskInDb;
  }
```

- Do not handle it in the controller and it will propagate to Nest which will be handled:

![](/md/217.jpg)

- Since `updateTaskStatus` uses `getTaskById` method, we don't have to add it there too. But in `deleteTask` method, we add `const found = this.getTaskById(id);` so to trigger the not found error.

## Custom Pipe Validating a Parameter

- Create `src/tasks/pipes/task-status-validation.pipe.ts`:

```ts
import {
  PipeTransform,
  ArgumentMetadata,
  BadRequestException,
} from "@nestjs/common";
import { TaskStatus } from "../task-status.enum";

export class TaskStatusValidationPipe implements PipeTransform {
  readonly allowedStatuses = [
    // readonly means it cannot be modified by class members
    TaskStatus.OPEN,
    TaskStatus.IN_PROGRESS,
    TaskStatus.DONE,
  ];

  transform(value: any, metadata: ArgumentMetadata) {
    // console.log(metadata); // { type: 'body', data: 'status', metatype: ...}

    value = value.toUpperCase();

    if (!this.isStatusValid(value)) {
      throw new BadRequestException(`"${value}" is an invalid status`);
    }

    return value;
  }

  private isStatusValid(status: any) {
    const idx = this.allowedStatuses.indexOf(status);
    return idx !== -1;
  }
}
```

- Then inside the controller, to apply it specifically fot `status` parameter:

```ts
@Patch("/:id/status")
  updateTaskStatus(
    @Param("id", ParseIntPipe) id: number, // change it to number -> defined in '@nestjs/common'
    @Body("status", TaskStatusValidationPipe) status: TaskStatus
  ): Task {
    return this.tasksService.updateTaskStatus(id, status);
  }
```

# TypeORM

![](/md/218.jpg)

![](/md/219.jpg)

![](/md/220.jpg)

![](/md/221.jpg)

![](/md/222.jpg)

- Install `npm i @nestjs/typeorm typeorm pg`.
- Create `src/config/typeorm.config.ts`:

```ts
import { TypeOrmModuleOptions } from "@nestjs/typeorm";

export const typeOrmConfig: TypeOrmModuleOptions = {
  type: "postgres",
  host: "localhost",
  port: 5432,
  username: "postgres",
  password: "postgres",
  database: "thenameofdb",
  autoLoadEntities: true, // all files which ends with .entity.ts will be picked up by TypeORM
  synchronize: true, // for production DO NOT set it to true -> It will sync the DB with the schema whenever it connects
};
```

- Change the `app.module`:

```ts
import { Module } from "@nestjs/common";
import { TypeOrmModule } from "@nestjs/typeorm";
import { typeOrmConfig } from "./config/typeorm.config";
import { TasksModule } from "./tasks/tasks.module";

@Module({
  imports: [TypeOrmModule.forRoot(typeOrmConfig), TasksModule],
})
export class AppModule {}
```

## Entity

- In TypeORM, we define entities that represent tables. We can have logic too.
- So we don't need `task.model` file. Remove it but keep the `TaskStatus` enum `task-status.enum.ts`.
- `src/tasks/task.entity.ts`:

```ts
import { BaseEntity, Entity, PrimaryGeneratedColumn, Column } from "typeorm";
import { TaskStatus } from "./task-status.enum";

@Entity()
export class Task extends BaseEntity {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column()
  description: string;

  @Column()
  status: TaskStatus;
}
```

## Repository

- We define repositories for our entities.
- Query builder is useful to build queries conditionally.
- Repositories will be called in our services, when we want to do database operations.
  `src/tasks/task.repository.ts`:

```ts
import { InternalServerErrorException } from "@nestjs/common";
import { EntityRepository, Repository } from "typeorm";
import { Task } from "./task.entity";
import { CreateTaskDto } from "./dto/create-task.dto";
import { GetTasksFilterDto } from "./dto/get-tasks-filter.dto";
import { TaskStatus } from "./task-status.enum";

@EntityRepository(Task)
export class TaskRepository extends Repository<Task> {
  async getTasks(filterDto: GetTasksFilterDto): Promise<Task[]> {
    const { status, search } = filterDto;
    const query = this.createQueryBuilder("task"); // 'task' is the keyword that we will use to refer to Task entity

    if (status) {
      query.andWhere("task.status = :status", { status }); // where will override but andWhere will add
    }

    if (search) {
      query.andWhere(
        "(task.title LIKE :search OR task.description LIKE :search)",
        { search: `%${search}%` }
      );
    }

    try {
      const tasks = await query.getMany();
      return tasks;
    } catch (error) {
      throw new InternalServerErrorException();
    }
  }

  async createTask(createTaskDto: CreateTaskDto): Promise<Task> {
    const { title, description } = createTaskDto;

    const task = new Task();
    task.title = title;
    task.description = description;
    task.status = TaskStatus.OPEN;

    try {
      await task.save();
    } catch (error) {
      throw new InternalServerErrorException();
    }

    return task;
  }
}
```

- Then we should inject our `TaskRepository` in `tasks.module`:

```ts
import { Module } from "@nestjs/common";
import { TasksController } from "./tasks.controller";
import { TasksService } from "./tasks.service";
import { TypeOrmModule } from "@nestjs/typeorm";
import { TaskRepository } from "./task.repository";

@Module({
  imports: [TypeOrmModule.forFeature([TaskRepository])], // we have to pass array of entities or repositories available for this module
  controllers: [TasksController],
  providers: [TasksService],
})
export class TasksModule {}
```

## Changes in Tasks Service

- Then change our `tasks.service`:

```ts
import { Injectable, NotFoundException } from "@nestjs/common";
import { InjectRepository } from "@nestjs/typeorm";
import { Task } from "./task.entity";
import { TaskRepository } from "./task.repository";
import { TaskStatus } from "./task-status.enum";
import { CreateTaskDto } from "./dto/create-task.dto";
import { GetTasksFilterDto } from "./dto/get-tasks-filter.dto";

@Injectable()
export class TasksService {
  constructor(
    @InjectRepository(TaskRepository) // for injecting repository we need this decorator as well
    private taskRepository: TaskRepository
  ) {}

  async getTasks(filterDto: GetTasksFilterDto): Promise<Task[]> {
    return this.taskRepository.getTasks(filterDto);
  }

  async getTaskById(id: number): Promise<Task> {
    const found = await this.taskRepository.findOne(id); // My opinion: We should have defined a method in repository. Maybe unless it is only one line.

    if (!found) {
      throw new NotFoundException(`Task with ID "${id}" not found`);
    }

    return found;
  }

  async createTask(createTaskDto: CreateTaskDto): Promise<Task> {
    return this.taskRepository.createTask(createTaskDto);
  }

  async deleteTask(id: number): Promise<void> {
    const result = await this.taskRepository.delete(id); // We have another method: 'remove'. Which we need to pass the task object to it. So we have to first retrieve it and then pass it as argument.
    // So delete method has fewer database calls

    if (result.affected === 0) {
      throw new NotFoundException(`Task with ID "${id}" not found`);
    }
  }

  async updateTaskStatus(id: number, status: TaskStatus): Promise<Task> {
    const task = await this.getTaskById(id);
    task.status = status;
    await task.save();
    return task;
  }
}
```

## Changes in Tasks Controller

- Then, change `tasks.controller`:

```ts
import {
  Controller,
  Get,
  Post,
  Body,
  Param,
  Delete,
  Patch,
  Query,
  UsePipes,
  ValidationPipe,
  ParseIntPipe,
  UseGuards,
} from "@nestjs/common";
import { TasksService } from "./tasks.service";
import { TaskStatusValidationPipe } from "./pipes/task-status-validation.pipe";
import { Task } from "./task.entity";
import { TaskStatus } from "./task-status.enum";
import { CreateTaskDto } from "./dto/create-task.dto";
import { GetTasksFilterDto } from "./dto/get-tasks-filter.dto";

@Controller("tasks")
export class TasksController {
  constructor(private tasksService: TasksService) {}

  @Get()
  getTasks(
    @Query(ValidationPipe) filterDto: GetTasksFilterDto
  ): Promise<Task[]> {
    return this.tasksService.getTasks(filterDto);
  }

  @Get("/:id")
  getTaskById(@Param("id", ParseIntPipe) id: number): Promise<Task> {
    return this.tasksService.getTaskById(id);
  }

  @Post()
  @UsePipes(ValidationPipe)
  createTask(@Body() createTaskDto: CreateTaskDto): Promise<Task> {
    return this.tasksService.createTask(createTaskDto);
  }

  @Delete("/:id")
  deleteTask(@Param("id", ParseIntPipe) id: number): Promise<void> {
    return this.tasksService.deleteTask(id);
  }

  @Patch("/:id/status")
  updateTaskStatus(
    @Param("id", ParseIntPipe) id: number,
    @Body("status", TaskStatusValidationPipe) status: TaskStatus
  ): Promise<Task> {
    return this.tasksService.updateTaskStatus(id, status);
  }
}
```

- So in general, they can be same method (like `createTask`) in handler -> service -> repository!

# Authentication

- We should create an `auth` module, controller, and service. `nest g module auth`, `nest g controller auth --no-spec`, `nest g service auth --no-spec`.
- Create `user.entity.ts`:

```ts
import {
  BaseEntity,
  Entity,
  PrimaryGeneratedColumn,
  Column,
  Unique,
  OneToMany,
} from "typeorm";
import * as bcrypt from "bcryptjs";
import { Task } from "../tasks/task.entity";

@Entity()
@Unique(["username"])
export class User extends BaseEntity {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  username: string;

  @Column()
  password: string;

  @Column()
  salt: string; // with bcrypt, we have to store salt in db as well? I thought it would be concatenated to the hashed password. Yes it will! Ariel is stupid

  @OneToMany((type) => Task, (task) => task.user, { eager: true }) // the second parameter is to define how to reach the user so we have to define user on task entity
  // Also, eager option means when loading a user, loads their tasks
  tasks: Task[];

  async validatePassword(password: string): Promise<boolean> {
    // it should be method on User entity
    const hash = await bcrypt.hash(password, this.salt);
    return hash === this.password;
  }
}
```

- Create `src/auth/dto/auth-credentials.dto.ts`:

```ts
import { IsString, MinLength, MaxLength, Matches } from "class-validator";

export class AuthCredentialsDto {
  @IsString()
  @MinLength(4)
  @MaxLength(20)
  username: string;

  @IsString()
  @MinLength(8)
  @MaxLength(20)
  @Matches(/((?=.*\d)|(?=.*\W+))(?![.\n])(?=.*[A-Z])(?=.*[a-z]).*$/, {
    message: "password too weak",
  })
  password: string;
}
```

- Now create the `user.repository`:

```ts
import { Repository, EntityRepository } from "typeorm";
import {
  ConflictException,
  InternalServerErrorException,
} from "@nestjs/common";
import * as bcrypt from "bcryptjs";
import { User } from "./user.entity";
import { AuthCredentialsDto } from "./dto/auth-credentials.dto";

@EntityRepository(User)
export class UserRepository extends Repository<User> {
  async signUp(authCredentialsDto: AuthCredentialsDto): Promise<void> {
    const { username, password } = authCredentialsDto;

    const user = new User();
    user.username = username;
    user.salt = await bcrypt.genSalt(); // it is better to generate a salt for each user
    user.password = await this.hashPassword(password, user.salt);

    try {
      await user.save();
    } catch (error) {
      if (error.code === "23505") {
        // this way, we only send one request to the database but we have to make sure that we have defined a unique constraint on the User entity
        // duplicate username
        throw new ConflictException("Username already exists");
      } else {
        throw new InternalServerErrorException(); // By default, if an error is not been handled, it will bubble up and create a 500 error so here we have intervened and then again re-thrown the error
      }
    }
  }

  async validateUserPassword(
    authCredentialsDto: AuthCredentialsDto
  ): Promise<string> {
    const { username, password } = authCredentialsDto;
    const user = await this.findOne({ username });

    if (user && (await user.validatePassword(password))) {
      return user.username;
    } else {
      return null;
    }
  }

  private async hashPassword(password: string, salt: string): Promise<string> {
    return bcrypt.hash(password, salt);
  }
}
```

- We are also using `passport.js` which is an authentication middleware for node.js and has different strategies which we will use `jwt`. It will authenticate our user from the token he has provided and add the user entity into request. `npm i @nestjs/jwt @nestjs/passport passport passport-jwt`.
- We need to import `JwtModule` into our auth module, because it has exported a service `JwtService` which can be injected in our auth service.
- Now import the `UserRepository`, `PassportModule`, and `JwtModule` in the `auth` module.
- Also, `JwtStrategy` (that we will write) is some kind of service to authenticate the requests and populate user property on them. So, we will export it as well (along with PassportModule).

```ts
import { Module } from "@nestjs/common";
import { TypeOrmModule } from "@nestjs/typeorm";
import { JwtModule } from "@nestjs/jwt";
import { PassportModule } from "@nestjs/passport";
import { AuthController } from "./auth.controller";
import { AuthService } from "./auth.service";
import { UserRepository } from "./user.repository";
import { JwtStrategy } from "./jwt.strategy";

const jwtConfig = config.get("jwt");

@Module({
  imports: [
    PassportModule.register({ defaultStrategy: "jwt" }),
    JwtModule.register({
      secret: "mySecret",
      signOptions: {
        expiresIn: 3600,
      },
    }),
    TypeOrmModule.forFeature([UserRepository]),
  ],
  controllers: [AuthController],
  providers: [AuthService, JwtStrategy],
  exports: [JwtStrategy, PassportModule],
})
export class AuthModule {}
```

- Now let's inject the repository in `auth.service`:

```ts
import { Injectable, UnauthorizedException } from "@nestjs/common";
import { JwtService } from "@nestjs/jwt";
import { InjectRepository } from "@nestjs/typeorm";
import { UserRepository } from "./user.repository";
import { AuthCredentialsDto } from "./dto/auth-credentials.dto";
import { JwtPayload } from "./jwt-payload.interface";

@Injectable()
export class AuthService {
  constructor(
    @InjectRepository(UserRepository)
    private userRepository: UserRepository,
    private jwtService: JwtService
  ) {}

  async signUp(authCredentialsDto: AuthCredentialsDto): Promise<void> {
    return this.userRepository.signUp(authCredentialsDto);
  }

  async signIn(
    authCredentialsDto: AuthCredentialsDto
  ): Promise<{ accessToken: string }> {
    const username = await this.userRepository.validateUserPassword(
      authCredentialsDto
    );

    if (!username) {
      throw new UnauthorizedException("Invalid credentials");
    }

    const payload: JwtPayload = { username }; // it is just an interface we have defined: export interface JwtPayload { username: string; }
    const accessToken = await this.jwtService.sign(payload);

    return { accessToken };
  }
}
```

- Now let's inject the `auth.service` into `auth.controller`:

```ts
import {
  Controller,
  Post,
  Body,
  ValidationPipe,
  UseGuards,
  Req,
} from "@nestjs/common";
import { AuthGuard } from "@nestjs/passport";
import { AuthCredentialsDto } from "./dto/auth-credentials.dto";
import { AuthService } from "./auth.service";

@Controller("auth")
export class AuthController {
  constructor(private authService: AuthService) {}

  @Post("/signup")
  signUp(
    @Body(ValidationPipe) authCredentialsDto: AuthCredentialsDto
  ): Promise<void> {
    return this.authService.signUp(authCredentialsDto);
  }

  @Post("/signin")
  signIn(
    @Body(ValidationPipe) authCredentialsDto: AuthCredentialsDto
  ): Promise<{ accessToken: string }> {
    return this.authService.signIn(authCredentialsDto);
  }

  @Post("/test")
  @UseGuards(AuthGuard()) // we can use guarding on the controller level or method level
  test(@Req() req) {
    console.log(req); // you can find the user property in the request
  }
}
```

# Authorization

- Create `src/auth/jwt.strategy.ts`:

```ts
import { Injectable, UnauthorizedException } from "@nestjs/common";
import { PassportStrategy } from "@nestjs/passport";
import { Strategy, ExtractJwt } from "passport-jwt";
import { JwtPayload } from "./jwt-payload.interface";
import { InjectRepository } from "@nestjs/typeorm";
import { UserRepository } from "./user.repository";
import { User } from "./user.entity";
import * as config from "config";

@Injectable() // Reminder: we usually annotate services as injectable to inject them in controllers
// but here, it will be a provider (we have to include it in providers array)
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor(
    @InjectRepository(UserRepository)
    private userRepository: UserRepository
  ) {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      secretOrKey: "mySecret",
    });
  }

  async validate(payload: JwtPayload): Promise<User> {
    // Remember that WE defined the JwtPayload
    // This method will be executed after jwt has been verified
    // anything returned from this method, will be added to request
    const { username } = payload;
    const user = await this.userRepository.findOne({ username });

    if (!user) {
      throw new UnauthorizedException();
    }

    return user;
  }
}
```

- We create a **custom decorator** `src/auth/get-user.decorator.ts` because we don't want to drill into req to get user in `tasks.controller` for example:

```ts
import { createParamDecorator, ExecutionContext } from "@nestjs/common";
import { User } from "./user.entity";

export const GetUser = createParamDecorator(
  (data, ctx: ExecutionContext): User => {
    //there will be no data as the first parameter
    const req = ctx.switchToHttp().getRequest();
    return req.user;
  }
);
```

- Then we can use it like `@GetUser() user: User` on action methods' parameters.
- To introduce authentication on tasks, we should import the `auth module` into the `tasks module` (because we need `JwtStrategy provider` of that module which has been exported):

```ts
import { Module } from "@nestjs/common";
import { TypeOrmModule } from "@nestjs/typeorm";
import { AuthModule } from "../auth/auth.module";
import { TasksController } from "./tasks.controller";
import { TasksService } from "./tasks.service";
import { TaskRepository } from "./task.repository";

@Module({
  imports: [TypeOrmModule.forFeature([TaskRepository]), AuthModule],
  controllers: [TasksController],
  providers: [TasksService],
})
export class TasksModule {}
```

- Then, we are able to use `AuthGuard` from Passport:

```ts
@Controller("tasks")
@UseGuards(AuthGuard())
export class TasksController {}
```

- We have to change `task.entity` to connect it to User:

```ts
import {
  BaseEntity,
  Entity,
  PrimaryGeneratedColumn,
  Column,
  ManyToOne,
} from "typeorm";
import { TaskStatus } from "./task-status.enum";
import { User } from "../auth/user.entity";

@Entity()
export class Task extends BaseEntity {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column()
  description: string;

  @Column()
  status: TaskStatus;

  @ManyToOne((type) => User, (user) => user.tasks, { eager: false })
  // we don't want to load user when loading a task
  user: User;

  @Column() // we still need to define this (although TypeORM will define it for us [because of the relationship above] but we want to get access to it inside code)
  userId: number;
}
```

- So we modify `tasks.controller` to get the user from req and pass it to `tasks.service`:

```ts
import {
  Controller,
  Get,
  Post,
  Body,
  Param,
  Delete,
  Patch,
  Query,
  UsePipes,
  ValidationPipe,
  ParseIntPipe,
  UseGuards,
} from "@nestjs/common";
import { AuthGuard } from "@nestjs/passport";
import { TasksService } from "./tasks.service";
import { CreateTaskDto } from "./dto/create-task.dto";
import { GetTasksFilterDto } from "./dto/get-tasks-filter.dto";
import { TaskStatusValidationPipe } from "./pipes/task-status-validation.pipe";
import { TaskStatus } from "./task-status.enum";
import { Task } from "./task.entity";
import { User } from "../auth/user.entity";
import { GetUser } from "../auth/get-user.decorator";

@Controller("tasks")
@UseGuards(AuthGuard())
export class TasksController {
  constructor(private tasksService: TasksService) {}

  @Get()
  getTasks(
    @Query(ValidationPipe) filterDto: GetTasksFilterDto,
    @GetUser() user: User
  ): Promise<Task[]> {
    return this.tasksService.getTasks(filterDto, user);
  }

  @Get("/:id")
  getTaskById(
    @Param("id", ParseIntPipe) id: number,
    @GetUser() user: User
  ): Promise<Task> {
    return this.tasksService.getTaskById(id, user);
  }

  @Post()
  @UsePipes(ValidationPipe)
  createTask(
    @Body() createTaskDto: CreateTaskDto,
    @GetUser() user: User
  ): Promise<Task> {
    return this.tasksService.createTask(createTaskDto, user);
  }

  @Delete("/:id")
  deleteTask(
    @Param("id", ParseIntPipe) id: number,
    @GetUser() user: User
  ): Promise<void> {
    return this.tasksService.deleteTask(id, user);
  }

  @Patch("/:id/status")
  updateTaskStatus(
    @Param("id", ParseIntPipe) id: number,
    @Body("status", TaskStatusValidationPipe) status: TaskStatus,
    @GetUser() user: User
  ): Promise<Task> {
    return this.tasksService.updateTaskStatus(id, status, user);
  }
}
```

- Then we modify `tasks.service` to receive the user and pass it to `tasks.repository`:

```ts
import { Injectable, NotFoundException } from "@nestjs/common";
import { InjectRepository } from "@nestjs/typeorm";
import { CreateTaskDto } from "./dto/create-task.dto";
import { GetTasksFilterDto } from "./dto/get-tasks-filter.dto";
import { TaskRepository } from "./task.repository";
import { TaskStatus } from "./task-status.enum";
import { Task } from "./task.entity";
import { User } from "../auth/user.entity";

@Injectable()
export class TasksService {
  constructor(
    @InjectRepository(TaskRepository)
    private taskRepository: TaskRepository
  ) {}

  async getTasks(filterDto: GetTasksFilterDto, user: User): Promise<Task[]> {
    return this.taskRepository.getTasks(filterDto, user);
  }

  async getTaskById(id: number, user: User): Promise<Task> {
    const found = await this.taskRepository.findOne({
      where: { id, userId: user.id },
    });

    if (!found) {
      throw new NotFoundException(`Task with ID "${id}" not found`);
    }

    return found;
  }

  async createTask(createTaskDto: CreateTaskDto, user: User): Promise<Task> {
    return this.taskRepository.createTask(createTaskDto, user);
  }

  async deleteTask(id: number, user: User): Promise<void> {
    const result = await this.taskRepository.delete({ id, userId: user.id }); // note that it is a bit different from findOne

    if (result.affected === 0) {
      throw new NotFoundException(`Task with ID "${id}" not found`);
    }
  }

  async updateTaskStatus(
    id: number,
    status: TaskStatus,
    user: User
  ): Promise<Task> {
    const task = await this.getTaskById(id, user);
    task.status = status;
    await task.save();
    return task;
  }
}
```

- and finally, `tasks.repository`:

```ts
import { InternalServerErrorException } from "@nestjs/common";
import { EntityRepository, Repository } from "typeorm";
import { CreateTaskDto } from "./dto/create-task.dto";
import { GetTasksFilterDto } from "./dto/get-tasks-filter.dto";
import { TaskStatus } from "./task-status.enum";
import { Task } from "./task.entity";
import { User } from "../auth/user.entity";

@EntityRepository(Task)
export class TaskRepository extends Repository<Task> {
  async getTasks(filterDto: GetTasksFilterDto, user: User): Promise<Task[]> {
    const { status, search } = filterDto;
    const query = this.createQueryBuilder("task");

    query.where("task.userId = :userId", { userId: user.id });

    if (status) {
      query.andWhere("task.status = :status", { status });
    }

    if (search) {
      query.andWhere(
        "(task.title LIKE :search OR task.description LIKE :search)",
        { search: `%${search}%` }
      );
    }

    try {
      const tasks = await query.getMany();
      return tasks;
    } catch (error) {
      throw new InternalServerErrorException();
    }
  }

  async createTask(createTaskDto: CreateTaskDto, user: User): Promise<Task> {
    const { title, description } = createTaskDto;

    const task = new Task();
    task.title = title;
    task.description = description;
    task.status = TaskStatus.OPEN;
    task.user = user; // nav prop

    try {
      await task.save();
    } catch (error) {
      throw new InternalServerErrorException();
    }

    delete task.user; //
    return task;
  }
}
```

# Logging

![](/md/223.jpg)

![](/md/224.jpg)

- You can define a logger as a private member in class:

```ts
export class TasksController {
  private logger = new Logger("TasksController");
  // Logger is in 'nestjs/common'
  // Argument is just a context (keyword) to group logs
  //...

  this.logger.verbose(`....`);
}
```

- Also, for logging errors, you should catch the error, log it (`this.logger.error(`...`, err.stack);`) and then re-throw the error. The second argument (stack trace) is helpful for debugging.

# Configuration Management

![](/md/225.jpg)

![](/md/226.jpg)

- You can use `config` package to manage code-base configuration based on environment by defining JSON/YML files in the `config` folder (next to `src`) and then use `config.get("key")` to get the value.
- For sensitive data you can use environment variables, you can use a combination like `process.env.JWT_SECRET || config.get('jwt').secret`.
- Also note that, it is not just about secrets, for changing without deployment we need environment variables (by the method above it will be fallback).
- You can easily enable CORS:

```ts
if (process.env.NODE_ENV === "development") {
  app.enableCors();
} else {
  app.enableCors({ origin: serverConfig.origin });
}
```

- Note that you have to change `"start:dev": "NODE_ENV=development nodemon",` in package.json.
- Also, note that `process.env.RDS_USERNAME`, ... is being set by `Elastic Beanstalk`.

# Stephen

## Basics

- A Nest app must have one module (`AppModule`) and one controller at minimum.
- When the Nest App starts up, it will create an instance of our controller classes.
- We will call `NestFactory` from `@nestjs/core` and give it the `AppModule` which creates an app which we will listen in the `main.ts`.
- We have class decorators, method decorators and parameter decorators such as:

![](/md/242.jpg)

- To setup pipes:

![](/md/243.jpg)

- The first step of the above image is by using `app.useGlobalPipes(new ValidationPipe({whitelist:true}));` in the `main.ts` before listening. So we don't need to use `@UsePipes(ValidationPipe)` in the handler level. Note that, the whitelist option will strip any extra data (in comparison to our DTO) that has been sent. It is a security improvement.

- `"emitDecoratorMetadata": true,` in `tsconfig.json` will leak some metadata from the DTO class type to the JS world which makes `class-transformer` to create an instance out of a plain object and then `class-validator` validates that instance.

![](/md/244.jpg)

- `Inversion of Control Principle`: Classes should not create instances of their dependencies on their own -> Better: we can inject in the constructor -> Best: we can inject it in the constructor but tie it to an interface. Excellent for testing.
  - In nest, we will use better approach and not the best. But we will see some workarounds.
  - Also, by default, the injected instance will be shared and it is persistent during the lifetime of the app (singleton) -> there are workarounds.

![](/md/245.jpg)

So:

![](/md/246.jpg)

![](/md/247.jpg)

- So you can export an item from the providers to exports: providers -> same module, exports -> other modules.
- If you want some service from a module, you should first import the module to another module. So you export a service but import a module.

## TypeORM

![](/md/248.jpg)

- So import the `TypeOrmModule` from `nestjs/typeorm` into the app module and configure it `imports: [TypeOrmModule.forRoot({...}), ...],`.

![](/md/249.jpg)

- So import the `TypeOrmModule` from `nestjs/typeorm` into a module along side with the entity and configure it `imports: [TypeOrmModule.forFeature([OurEntity]), ...],`. This creates a respoitory.
- Then import `OurEntity` into the app module and pass it in the `entities` array prop as configuring the TypeORM. It will create a repository automatically for us (?!).

- `synchronize:true` is meant to be used only for development and makes our schema migrations (change structure of database) to be reflected immediately (so we don't need to write any migration file). It is atypical behavior and most ORMs do not behave like this.
- In TypeORM, there are many ways to do a single task.
- Unlike the Ariel's way, we don't create another repo and instead, we inject the repo in the service by something like:

```ts
import { Injectable } from "nestjs/common";
import { Repository } from "typeorm";
import { InjectRepository } from "nestjs/typeorm";
import { User } from "./user.entity";

@Injectable()
export class UsersService {
  constructor(@InjectRepository(User) private usersRepo: Repository<User>) {}
}
```

- Note that because dependency injection is not working well with generics, we have to add that InjectRepository decorator.
- The `userRepo` is an abstraction over the table and has methods to `const user = userRepo.create({email, password})`, `return userRepo.save(user);`, `userRepo.find()`, `userRepo.findOne()`, `userRepo.remove()`...
- We could instantiate a new instance of the entity class, modify its props, and then call save method on it to make it persisted. But using create on the repository is preferred to create a new instance because otherwise we have to define a constructor for the entity or assign the properties one by one (which Ariel did).
- Note that we could directly save new users in the UsersService (`return userRepo.save({email, password});`), but we don't. Because, we might do some validation in the UsersService (apart from DTO). So we always create an instance then we save it.
- Also if we wanted to use some hooks in the entity (some decorators in the typeorm package like: `AfterUpdate`, `AfterInsert`, ... We annotate some method in the entity to be executed after that event), they wouldn't run if we call save directly and we have to first create an instance and then call save on it.
- So on repository, `save()` and `remove()` will be called with entity instance and causes the hooks to be executed but with `insert()`, `update()`, or `delete()` which perform the task directly on table, the hooks won't be executed.
- Of course for `update` and `delete`, if we use `save` or `remove`, it will be a bit inefficient. Because we have to first fetch the data. But it is what it is.

- It is better not to throw http exceptions (but we will ;]) in the service because if we want to re-use our service with WebSocket controller, we will have a problem.

![](/md/250.jpg)

- To exclude a property when serialization (sending back the response as JSON), Nest suggests that importing `Exclude` decorator in the entity class and use it on the desired property. Then using `@UseInterceptors(ClassSerializerInterceptor)` (both defined in the `nestjs/common` module) decorator on the controller method.
- But this approach has a problem if we want two different routes use the same service and return different results:

![](/md/251.jpg)

## Interceptors

- So we will use this approach:

![](/md/252.jpg)

- Which is using custom `interceptor`. A custom interceptor is a class that messes around with a response before sending it back to the client by using an outgoing DTO (it can also mess around with incoming requests). We can have different DTOs for different routes. It is like `middleware`.
- Interceptors can be applied to a single handler, all the handlers in a controller or globally,

![](/md/253.jpg)

```ts
import {
  UseInterceptors,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
} from "@nestjs/common";
import { Observable } from "rxjs";
import { map } from "rxjs/operators";
import { plainToClass } from "class-transformer";

export class SerializeInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, handler: CallHandler): Observable<any> {
    // Run something before a request is handled
    // by the request handler
    console.log("Im running before the handler", context);

    return handler.handle().pipe(
      map((data: any) => {
        // Run something before the response is sent out
        console.log("Im running before response is sent out", data);
      })
    );
  }
}
```

- Then we use `@UseInterceptors(SerializeInterceptor)` on the route that we want.
- For serialization, we get the data as an entity instance and we have to map it to DTO (just like mapper in .Net).

![](/md/254.jpg)

- Note that we don't put any validation on the outgoing DTO. We just decorate it with `Expose` decorator from class-transformer, so Nest will know that it should be exposed when serializing to JSON.

```ts
import {
  UseInterceptors,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
} from "@nestjs/common";
import { Observable } from "rxjs";
import { map } from "rxjs/operators";
import { plainToClass } from "class-transformer";

interface ClassConstructor {
  // a bit type safety so we have to pass at least class to serializer
  // in general it is challenging to have type safety inside decorators in TS
  new (...args: any[]): {};
}

export function Serialize(dto: ClassConstructor) {
  return UseInterceptors(new SerializeInterceptor(dto));
}

export class SerializeInterceptor implements NestInterceptor {
  constructor(private dto: any) {}

  intercept(context: ExecutionContext, handler: CallHandler): Observable<any> {
    return handler.handle().pipe(
      map((data: any) => {
        return plainToClass(this.dto, data, {
          excludeExtraneousValues: true,
        });
      })
    );
  }
}
```

- Then we use `@Serialize(UserDto)` on the route that we want or on the controller to be reflected on all routes (if we had two routes with different DTOs, we would use it on routes and not on the controller level).
- Since the decorator was long, we shortened it with `Serialize` custom decorator.
- Decorators are plain functions.

## Authentication

![](/md/255.jpg)

- Install `cookie-session` and `@types/cookie-session`.

![](/md/256.jpg)

- Then, inside `main.ts`:

```ts
//...
const cookieSession = require("cookie-session");

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.use(
    cookieSession({
      keys: ["asdfasfd"], // it is like the secret
    })
  );
  //...
}
//...
```

- Then in your routes you can get the session from header and set it easily:

```ts
@Get("/colors/:color")
setColor(@Param("color") color:string, @Session() session: any) {
  session.color = color;
}

@Get("/colors")
getColor(@Session() session: any) {
  return session.color;
}
```

- So in signin and signup routes, we will set the userId to user.id on the session object.

![](/md/257.jpg)

- To get the full user object by a decorator, since the decorator is outside the DI system, the decorator cannot get an instance of the UsersService. So we create an interceptor to get the current user and then use the value produced by it in the decorator. So interceptor (like a middleware) will get the userId from the session object, makes a db call to retrieve the user, and populates the curretUser property of the request object. The decorator will only get the currentUser from the request and it is just a nice thing to have. Otherwise we had to add `@Request() request: Request` in the route handler parameter and use the currentuser from it.
- Very important: you will see that this is wrong because interceptors will run after guards and it doesn't work for AdminGuard. So we will change it to CurrentUser middleware.

```ts
import {
  NestInterceptor,
  ExecutionContext,
  CallHandler,
  Injectable,
} from "@nestjs/common";
import { UsersService } from "../users.service";

@Injectable()
export class CurrentUserInterceptor implements NestInterceptor {
  constructor(private usersService: UsersService) {}

  async intercept(context: ExecutionContext, handler: CallHandler) {
    const request = context.switchToHttp().getRequest();
    const { userId } = request.session || {};

    if (userId) {
      const user = await this.usersService.findOne(userId);
      request.currentUser = user;
    }

    return handler.handle();
  }
}
```

- We have to annotate it with Injectable because we want this interceptor to be part of DI system. Then, we have to list it inside providers arrays for that module. Then we can use this interceptor to annotate a controller (or a route handler). Note that you can only use the following decorator on routes that you have already used this interceptor on:

```ts
import { createParamDecorator, ExecutionContext } from "@nestjs/common";

export const CurrentUser = createParamDecorator(
  (data: never, context: ExecutionContext) => {
    const request = context.switchToHttp().getRequest();
    return request.currentUser;
  }
);
```

- We also can use that CurrentUserInterceptor globally in users module by changing the interceptor in the providers array to an object and removing using it on any controller or route:

```ts
import { Module } from "@nestjs/common";
import { APP_INTERCEPTOR } from "@nestjs/core";
import { TypeOrmModule } from "@nestjs/typeorm";
import { UsersController } from "./users.controller";
import { UsersService } from "./users.service";
import { AuthService } from "./auth.service";
import { User } from "./user.entity";
import { CurrentUserInterceptor } from "./interceptors/current-user.interceptor";

@Module({
  imports: [TypeOrmModule.forFeature([User])],
  controllers: [UsersController],
  providers: [
    UsersService,
    AuthService,
    {
      provide: APP_INTERCEPTOR,
      useClass: CurrentUserInterceptor,
    },
  ],
})
export class UsersModule {}
```

- Note that it is global and not limited to controllers defined in this module. Of course there might be some controllers or routes that don't care about the current user but it is what it is.

![](/md/258.jpg)

![](/md/259.jpg)

```ts
import { CanActivate, ExecutionContext } from "@nestjs/common";

export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext) {
    const request = context.switchToHttp().getRequest();

    return request.session.userId;
  }
}
```

- Then use this decorator: `@UseGuards(AuthGuard())`.
- As a general note the context is a http request but to be able to expanded to other comunications protocols (ws, gRPC, ...), Nest has introduced context.

## Test

You can dramatically speed up your tests by updating the package.json file.

In the scripts section, find the following line:

`"test:watch": "jest --watch",`

And change it to:

`"test:watch": "jest --watch --maxWorkers=1",`

Restart your test runner at your terminal after making this change.

- For example, for unit testing AuthService we will mock all of its dependencies and test the class in isolation.

```ts
import { Test } from "@nestjs/testing";
import { AuthService } from "./auth.service";
import { UsersService } from "./users.service";
import { User } from "./user.entity";

describe("AuthService", () => {
  let service: AuthService;
  let fakeUsersService: Partial<UsersService>;

  beforeEach(async () => {
    // Create a fake copy of the users service
    fakeUsersService = {
      find: () => Promise.resolve([]),
      create: (email: string, password: string) =>
        Promise.resolve({ id: 1, email, password } as User),
    };

    const module = await Test.createTestingModule({
      providers: [
        AuthService,
        {
          provide: UsersService,
          useValue: fakeUsersService,
        },
      ],
    }).compile();

    service = module.get(AuthService);
  });

  it("can create an instance of auth service", async () => {
    expect(service).toBeDefined();
  });

  it("creates a new user with a salted and hashed password", async () => {
    const user = await service.signup("asdf@asdf.com", "asdf");

    expect(user.password).not.toEqual("asdf");
    const [salt, hash] = user.password.split(".");
    expect(salt).toBeDefined();
    expect(hash).toBeDefined();
  });

  it("throws an error if user signs up with email that is in use", async (done) => {
    fakeUsersService.find = () =>
      Promise.resolve([{ id: 1, email: "a", password: "1" } as User]);
    try {
      await service.signup("asdf@asdf.com", "asdf");
    } catch (err) {
      done();
    }
  });
});
```

- To test a controller, note that we can't test the decorators so there is not much to test really (to test decorators, we have to write e2e tests):

```ts
import { Test, TestingModule } from "@nestjs/testing";
import { UsersController } from "./users.controller";
import { UsersService } from "./users.service";
import { AuthService } from "./auth.service";
import { User } from "./user.entity";

describe("UsersController", () => {
  let controller: UsersController;
  let fakeUsersService: Partial<UsersService>;
  let fakeAuthService: Partial<AuthService>;

  beforeEach(async () => {
    fakeUsersService = {
      findOne: (id: number) => {
        return Promise.resolve({
          id,
          email: "asdf@asdf.com",
          password: "asdf",
        } as User);
      },
      find: (email: string) => {
        return Promise.resolve([{ id: 1, email, password: "asdf" } as User]);
      },
      // remove: () => {},
      // update: () => {},
    };
    fakeAuthService = {
      // signup: () => {},
      signin: (email: string, password: string) => {
        return Promise.resolve({ id: 1, email, password } as User);
      },
    };

    const module: TestingModule = await Test.createTestingModule({
      controllers: [UsersController],
      providers: [
        {
          provide: UsersService,
          useValue: fakeUsersService,
        },
        {
          provide: AuthService,
          useValue: fakeAuthService,
        },
      ],
    }).compile();

    controller = module.get<UsersController>(UsersController);
  });

  it("should be defined", () => {
    expect(controller).toBeDefined();
  });

  it("findAllUsers returns a list of users with the given email", async () => {
    const users = await controller.findAllUsers("asdf@asdf.com");
    expect(users.length).toEqual(1);
    expect(users[0].email).toEqual("asdf@asdf.com");
  });

  it("findUser returns a single user with the given id", async () => {
    const user = await controller.findUser("1");
    expect(user).toBeDefined();
  });

  it("findUser throws an error if user with given id is not found", async (done) => {
    fakeUsersService.findOne = () => null;
    try {
      await controller.findUser("1");
    } catch (err) {
      done();
    }
  });

  it("signin updates session object and returns user", async () => {
    const session = { userId: -10 };
    const user = await controller.signin(
      { email: "asdf@asdf.com", password: "asdf" },
      session
    );

    expect(user.id).toEqual(1);
    expect(session.userId).toEqual(1);
  });
});
```

- For running unit tests, we run `npm run test:watch` and for integration tests, run `npm run test:e2e`.
- Note that for `e2e`, the main.ts won't run and so our cookie-session middleware and validation pipe are not running.
- An easy solution would be create a helper function (setupApp(app: any)=>{...}) to setup those middlewares and pipes and then import it in the two files.
- But the Nest official solution is that you move the global setup to app module and not in main.ts.

![](/md/260.jpg)

```ts
import { Test, TestingModule } from "@nestjs/testing";
import { INestApplication } from "@nestjs/common";
import * as request from "supertest";
import { AppModule } from "./../src/app.module";

describe("Authentication System", () => {
  let app: INestApplication;

  beforeEach(async () => {
    const moduleFixture: TestingModule = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = moduleFixture.createNestApplication();
    await app.init();
  });

  it("handles a signup request", () => {
    const email = "asdlkq4321@akl.com";

    return request(app.getHttpServer())
      .post("/auth/signup")
      .send({ email, password: "alskdfjl" })
      .expect(201)
      .then((res) => {
        const { id, email } = res.body;
        expect(id).toBeDefined();
        expect(email).toEqual(email);
      });
  });

  it("signup as a new user then get the currently logged in user", async () => {
    const email = "asdf@asdf.com";

    const res = await request(app.getHttpServer())
      .post("/auth/signup")
      .send({ email, password: "asdf" })
      .expect(201);

    const cookie = res.get("Set-Cookie"); //because supertest does not respect cookie

    const { body } = await request(app.getHttpServer())
      .get("/auth/whoami")
      .set("Cookie", cookie)
      .expect(200);

    expect(body.email).toEqual(email);
  });
});
```

- To help change the database in test env, we will create a Config service and we will inject it in the app module. To do that `npm i @nestjs/config`. Under the hood, this library uses `dotenv` package to set the env variable from files. Stephen says why we don't use the dotenv directly and why we need a service. Anyway, we need to inject it:

```ts
import { Module, ValidationPipe, MiddlewareConsumer } from "@nestjs/common";
import { APP_PIPE } from "@nestjs/core";
import { TypeOrmModule } from "@nestjs/typeorm";
import { ConfigModule, ConfigService } from "@nestjs/config";
import { AppController } from "./app.controller";
import { AppService } from "./app.service";
import { UsersModule } from "./users/users.module";
import { ReportsModule } from "./reports/reports.module";
import { User } from "./users/user.entity";
import { Report } from "./reports/report.entity";
const cookieSession = require("cookie-session");

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      envFilePath: `.env.${process.env.NODE_ENV}`,
    }),
    TypeOrmModule.forRootAsync({
      inject: [ConfigService],
      useFactory: (config: ConfigService) => {
        return {
          type: "sqlite",
          database: config.get<string>("DB_NAME"),
          synchronize: true,
          entities: [User, Report],
        };
      },
    }),
    // TypeOrmModule.forRoot({
    //   type: 'sqlite',
    //   database: 'db.sqlite',
    //   entities: [User, Report],
    //   synchronize: true,
    // }),
    UsersModule,
    ReportsModule,
  ],
  controllers: [AppController],
  providers: [
    AppService,
    {
      provide: APP_PIPE,
      useValue: new ValidationPipe({
        whitelist: true,
      }),
    },
  ],
})
export class AppModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(
        cookieSession({
          keys: ["asdfasfd"],
        })
      )
      .forRoutes("*");
  }
}
```

- Now we have to make sure to set the environment correctly by changing package.json. For non-windows, we could just the environment before the command. But for windows or in general we can use cross-env library to set the env variable before starting in dev.
- Note that Jest runs tests in parallel but then the test database might not like to have parallel connection so make it one test at a time by adding that `--maxWorkers=1` flag.
- To reset database, we will delete DB in global beforeEach (and disconnect in global afterEach) and rely on Nest to recreate DB for us with that sync option.

## Association

![](/md/261.jpg)

- `ManyToOne` decorator will cause a change (it adds a new column).
- `OneToMany` decorator does not change the db.

![](/md/262.jpg)

- Of course the association is not loaded automatically and we have to set some options when querying (or use eager loading).
- The first argument to OneToMany is a function because of the circular dependency that present in TS when relating classes so we define the type by function and not directly.

![](/md/263.jpg)

![](/md/264.jpg)

- So we will change CurrentUserInterceptor to CurrentUserMiddleware in the UsersModule but we wire it globally inside that UsersModule. The reason that we don't move it to app module is that then we have to export the UsersService and then the order can be mixed up as well. Because Cookie-Session middleware is also in app module and should run before this one. But note that Guards will be defined in root folder.

## Query Builder

- To create a complex query, use query builder in TypeORM.

```ts
 createEstimate({ make, model, lng, lat, year, mileage }: GetEstimateDto) {
    return this.repo
      .createQueryBuilder()
      .select('AVG(price)', 'price')
      .where('make = :make', { make })
      .andWhere('model = :model', { model })
      .andWhere('lng - :lng BETWEEN -5 AND 5', { lng })
      .andWhere('lat - :lat BETWEEN -5 AND 5', { lat })
      .andWhere('year - :year BETWEEN -3 AND 3', { year })
      .andWhere('approved IS TRUE')
      .orderBy('ABS(mileage - :mileage)', 'DESC')
      .setParameters({ mileage })
      .limit(3)
      .getRawOne();
  }
```

## Migrations

- We shouldn't use synchronize flag in production and instead we should use migration files.

![](/md/265.jpg)

![](/md/266.jpg)

- But the problem is that TypeORM does not know how to connect to DB (it is in App Module in Nest!). It does not know what a config service is.
- So we have to create the configurations as env variables but for Heroku, the DB env variables are not what TypeORM expects. So we have to define a `ormconfig.js` file in the root of the project:

```js
var dbConfig = {
  synchronize: false,
  migrations: ["migrations/*.js"],
  cli: {
    migrationsDir: "migrations",
  },
};

switch (process.env.NODE_ENV) {
  case "development":
    Object.assign(dbConfig, {
      type: "sqlite",
      database: "db.sqlite",
      entities: ["**/*.entity.js"],
    });
    break;
  case "test":
    Object.assign(dbConfig, {
      type: "sqlite",
      database: "test.sqlite",
      entities: ["**/*.entity.ts"],
      migrationsRun: true, // so we will run the migrations ... (like synchronize:true??)
    });
    break;
  case "production":
    break;
  default:
    throw new Error("unknown environment");
}

module.exports = dbConfig;
```

- For the test env to correctly read the js files, `allowJs: true` in `tsconfig.json`.
- Also exclude it in `tsconfig.build.json` like: `{ "extends": "./tsconfig.json", "exclude": ["ormconfig.js", "migrations", "node_modules", "test", "dist", "**/*spec.ts"] }`.

![](/md/267.jpg)

- To install `TypeORM CLI`, add `"typeorm": "cross-env NODE_ENV=development node --require ts-node/register ./node_modules/typeorm/cli.js"` in package.json file.
- Then when we want to add a migration `npm run typeorm migration:generate -- -n initial-schema -o`. `-o` is to make sure to generate a js file.
- To tun the migration `npm run typeorm migration:run`.
- Note that if there is any change, when generating the TypeORM will populate the migration based on the changes automatically but the auto-generated migration won't work properly when we move our app to production. Please replace the contents of your initial-schema file with the following:

```js
const { MigrationInterface, QueryRunner, Table } = require("typeorm");

module.exports = class initialSchema1625847615203 {
  name = "initialSchema1625847615203";

  async up(queryRunner) {
    await queryRunner.createTable(
      new Table({
        name: "user",
        columns: [
          {
            name: "id",
            type: "integer",
            isPrimary: true,
            isGenerated: true,
            generationStrategy: "increment",
          },
          {
            name: "email",
            type: "varchar",
          },
          {
            name: "password",
            type: "varchar",
          },
          {
            name: "admin",
            type: "boolean",
            default: "true",
          },
        ],
      })
    );

    await queryRunner.createTable(
      new Table({
        name: "report",
        columns: [
          {
            name: "id",
            type: "integer",
            isPrimary: true,
            isGenerated: true,
            generationStrategy: "increment",
          },
          { name: "approved", type: "boolean", default: "false" },
          { name: "price", type: "float" },
          { name: "make", type: "varchar" },
          { name: "model", type: "varchar" },
          { name: "year", type: "integer" },
          { name: "lng", type: "float" },
          { name: "lat", type: "float" },
          { name: "mileage", type: "integer" },
          { name: "userId", type: "integer" },
        ],
      })
    );
  }

  async down(queryRunner) {
    await queryRunner.query(`DROP TABLE ""report""`);
    await queryRunner.query(`DROP TABLE ""user""`);
  }
};
```
