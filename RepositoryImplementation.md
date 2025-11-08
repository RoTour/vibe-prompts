### **LLM Generation Prompt: DDD Repository Implementation**

**ROLE:** You are an expert software engineer specializing in TypeScript, Domain-Driven Design (DDD), and Clean Architecture. Your task is to generate concrete repository implementations based on a domain interface and its corresponding aggregate root entity.

**CONTEXT:** The project is a SvelteKit application using Prisma for the database ORM. Your generated code must strictly adhere to the architectural patterns, file structures, and coding conventions demonstrated in the provided reference files. The primary goal is to create a persistence layer that is decoupled from the domain.

**PRIMARY TASK:**

Given the file content for a domain repository interface (e.g., `I[AggregateName]Repository.ts`) and its corresponding domain aggregate root entity (e.g., `[AggregateName].entity.ts`), you must generate two complete, production-quality repository implementation files:

1.  `InMemory[AggregateName]Repository.ts`
2.  `Prisma[AggregateName]Repository.ts`

**DETAILED INSTRUCTIONS:**

**1. File and Folder Structure:**
   - Both implementation files must be created in the correct infrastructure folder: `src/[context]/infra/[AggregateName]Repository/`.
   - Replace `[context]` and `[AggregateName]` based on the provided domain files (e.g., for `IPromotionRepository`, the aggregate is `Promotion`).

**2. `InMemory[AggregateName]Repository.ts` Implementation:**
   - The class must implement the `I[AggregateName]Repository` interface.
   - Use a `private readonly Map<string, [AggregateName]>` for the in-memory storage.
   - The implementation should be straightforward, using standard `Map` operations (`get`, `set`, `values`) to fulfill the interface contract.
   - Ensure all necessary domain and interface types are imported correctly.

**3. `Prisma[AggregateName]Repository.ts` Implementation:**

   **3.1. Mapper Class:**
   - Define a private `[AggregateName]Mapper` class inside the file. This class is responsible for all transformations between the domain model and the Prisma model.
   - **`fromPrismaToDomain` static method:**
     - **Signature:** `static fromPrismaToDomain(prismaModel: Prisma[AggregateName]WithRelations): [AggregateName]`
     - It must accept a Prisma model object as input. The type for this object must be explicitly defined to include any relations required for re-hydration (e.g., `type Prisma[AggregateName]WithRelations = Prisma[AggregateName] & { students: ... }`).
     - It must call a static `rehydrate` method on the domain aggregate class to construct the domain object. This is critical for bypassing creation logic and enforcing consistency.
     - It must correctly instantiate all domain Value Objects (e.g., `new [AggregateName]Id(prismaModel.id)`) from the raw Prisma data.
   - **`fromDomainToPrisma` static method:**
     - **Signature:** `static fromDomainToPrisma(domainModel: [AggregateName]): Prisma.[AggregateName]CreateInput & { id: string }`
     - It must accept a domain `[AggregateName]` entity as input.
     - It must return an object matching the type Prisma expects for a `create` operation.
     - It must flatten all domain Value Objects into their primitive database types (e.g., `domainModel.id.id()`).
     - For many-to-many relations, it must generate the correct nested write structure using `create` and `connect` as demonstrated in the reference `PrismaPromotionRepository`.

   **3.2. Repository Class:**
   - The class must implement the `I[AggregateName]Repository` interface.
   - **Constructor:** It must accept a `Prisma.PrismaClient` for dependency injection. No default value shoud be provided. This ensures testability.
   - **`save` method:**
     - It must use `this.prisma.[aggregateName].upsert` to handle both creation and updates in a single, atomic operation.
     - The `where` clause must use the aggregate's ID.
     - The `create` and `update` clauses must be populated with data from the `[AggregateName]Mapper`.
     - For many-to-many relations, the `update` clause **must** first use `deleteMany: {}` to clear existing relations before using `create` to re-establish them, ensuring the database is perfectly synchronized with the state of the domain aggregate.
   - **`findById` / `findAll` methods:**
     - They must use `this.prisma.[aggregateName].findUnique` and `findMany` respectively.
     - They must use `include` to eagerly fetch all relational data required by the `fromPrismaToDomain` mapper. Use `select` within the `include` to be efficient where possible (e.g., `select: { studentId: true }`).
     - They must pass the raw Prisma result(s) through the mapper to return pure domain aggregate(s).

**4. Imports and Typing:**
   - All imports must be fully specified with correct paths.
   - All variables, function parameters, and return types must be explicitly and accurately typed. Pay close attention to types imported from `$prisma/client`, domain entities, and value objects.
   - Since recent versions of prisma, `@prisma/client` no longer exports types directly. You must import types from the generated Prisma client located at `$prisma/client` instead.
