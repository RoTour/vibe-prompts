# Prompt: Create a Query Interface and its Prisma Implementation

## Objective

Generate two files: a TypeScript interface for a query service and its concrete implementation using Prisma. This follows the principles of Clean Architecture, separating the application contract from the infrastructure details.

## Placeholders

Please replace the following placeholders before using this prompt:
- `[context]`: The name of the bounded context (e.g., `quiz-context`).
- `[QueryName]`: The name of the query (e.g., `StudentsOverview`).
- `[DTOs]`: A description of the Data Transfer Objects needed for the query results. Include property names and types.
- `[Methods]`: A description of the methods for the interface. For each method, specify:
    - Its name (e.g., `getStudentsFromPromotion`).
    - Its parameters, including their names and types (e.g., `promotionId: PromotionId`, `teacherId: TeacherId`).
    - Its return type (e.g., `Promise<StudentOverviewDTO[]>`).

---

### File 1: The Query Interface

**Action:** Create the following file.

**Path:** `src/[context]/application/interfaces/I[QueryName]Queries.ts`

**Content Instructions:**

1.  **Imports:**
    - Import any necessary domain Value Objects (e.g., `PromotionId`, `TeacherId`) from `$[context]/domain/...`.

2.  **DTOs:**
    - Based on the `[DTOs]` placeholder, define the necessary Data Transfer Objects. Use `export type` for simple data structures or `export class` if they have methods (e.g., for construction).
    - Example:
      ```typescript
      export type StudentOverviewDTO = {
          id: string;
          name: string;
          email: string | null;
          status: 'online' | 'offline';
      };
      ```

3.  **Interface Definition:**
    - Define and export the interface named `I[QueryName]Queries`.
    - Add the methods specified in the `[Methods]` placeholder to this interface, ensuring all parameters and return types are strongly typed.
    - Example:
      ```typescript
      export interface IStudentsOverviewQueries {
          getStudentsFromPromotion(promotionId: PromotionId, teacherId: TeacherId): Promise<StudentOverviewDTO[]>;
      }
      ```

---

### File 2: The Prisma Implementation

**Action:** Create the following file.

**Path:** `src/[context]/infra/queries/Prisma[QueryName]Queries.ts`

**Content Instructions:**

1.  **Imports:**
    - `import type { I[QueryName]Queries } from '$[context]/application/interfaces/I[QueryName]Queries';`
    - `import type { PrismaClient } from '$prisma/client';`
    - Import all necessary DTOs from the interface file.
    - Import any domain Value Objects used in method signatures.

2.  **Class Definition:**
    - Define and export a class `Prisma[QueryName]Queries`.
    - It must implement the corresponding interface: `implements I[QueryName]Queries`.

3.  **Constructor:**
    - Create a constructor that accepts a `PrismaClient` instance and stores it in a private readonly property.
    - `constructor(private readonly client: PrismaClient) {}`

4.  **Method Implementation:**
    - Implement all methods from the interface.
    - Methods must be `async`.
    - Use `this.client` to execute Prisma queries (`findMany`, `findUnique`, etc.).
    - When filtering by a domain Value Object, use its primitive value (e.g., `where: { id: entityId.id() }`).
    - **Crucially, map the raw Prisma result to the DTOs defined in the application layer.** Do not return raw Prisma models. This ensures the infrastructure layer is completely decoupled.
    - If you need related data, use Prisma's `include` or nested `where` clauses.
    - Example:
      ```typescript
      async getStudentsFromPromotion(promotionId: PromotionId, teacherId: TeacherId): Promise<StudentOverviewDTO[]> {
          const students = await this.client.student.findMany({
              where: {
                  promotionId: promotionId.id(),
                  promotion: {
                      teacherId: teacherId.id()
                  }
              }
          });

          return students.map((student) => ({
              id: student.id,
              name: student.name,
              email: student.email,
              status: 'offline' // Or some other logic
          }));
      }
      ```
