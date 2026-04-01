# 🚀 TypeScript Interview Questions - Scenario-Based Practice

This document contains practical, real-world scenarios for common TypeScript interview concepts. Use these examples to demonstrate your architectural understanding during interviews.

---

## 🟢 TypeScript Interview Questions for Freshers

### 1. Data Types
**🔥 Scenario (Banking Application):**
- `string`: Establishing the user's full legal name ("Saurabh").
- `number`: Managing the exact account fractional balance (`5000.50`).
- `boolean`: Storing verification flags, such as whether KYC is completed (`true`).
- `any`: Handling unstructured JSON payloads from legacy third-party payment gateways.
- `void`: A function that pops up a visual success toast notification on the screen but yields no data back.

### 2. Variable Declarations (let vs const)
**🔥 Scenario (React Dashboard Generation):**
- `const API_BASE_URL = "https://api.example.com"`: Used for definitive endpoints ensuring security and preventing accidental overwrites.
- `let unreadNotificationCount = 0`: Used for counters that increment as new data webhooks flow in.

### 3. Explicit Variables
**🔥 Scenario (Handling User Profiles on Netflix):**
```typescript
let userAge: number = 24;
// userAge = "25"; // Catching error early
```
If an API mapping accidentally attempts to pass `"24"` as a string, TypeScript will throw a compile-time error, averting frontend runtime crashes.

### 4. Typed Functions
**🔥 Scenario (E-commerce Cart Checkout - e.g., Flipkart):**
```typescript
function calculateTotal(price: number, quantity: number): number {
    return price * quantity;
}
```
If a glitch in the UI interface mistakenly passes a string input for `quantity`, the ecosystem isolates the error pre-deployment.

### 5. "any" Type
**🔥 Scenario (Stripe API Integration Setup):**
When initially exploring an undocumented external API where the exact response payload structure is obscured, engineers rely on `any` as a rapid temporary bridge to unblock logical development before drafting rigorous generic interfaces.

### 6. Advantages of TypeScript
**🔥 Scenario (Scale with LinkedIn or Microsoft Teams):**
In monolithic systems traversing 50,000+ files, foundational JavaScript errors like "Cannot read property of undefined" possess catastrophic production outcomes. TypeScript halts these deployments explicitly, leading to 5x faster bug resolutions and stable deployments.

### 8. Void Type
**🔥 Scenario (Push Notification Engine):**
```typescript
function dispatchPushNotification(deviceId: string): void {
    console.log(`Sending ping broadcast to ${deviceId}`);
}
```

### 9. Null Type
**🔥 Scenario (Database Session Retrieval):**
```typescript
function findActiveUserSession(sessionId: string): Session | null { ... }
```
If a database probe fails to extract the session hash, the architecture overtly returns `null`, compelling the frontend validation layer to gracefully render an "Unauthenticated Login" screen.

### 10-11. Objects & Optional Properties
**🔥 Scenario (Social Media Account Settings):**
```typescript
const userSettings: { username: string; age?: number } = {
    username: "Saurabh"
};
```
Newly onboarded users may not have supplied their age yet. Flagging it as optional (`?`) circumvents code breaks when iterating across user datasets.

### 12. Undefined
**🔥 Scenario (Multi-Step Web Forms):**
When managing a granular registration form where a user elects to bypass an optional "Middle Name" string field, the container variable endures in memory but remains formally `undefined` pending final form submission.

### 13. Arrays
**🔥 Scenario (Trello-like Task Management State):**
```typescript
let sprintTasks: string[] = ["Design Database Diagram", "Deploy Container"];
// sprintTasks.push(404); // Blocks injecting invalid primitive types
```

### 14-15. Compiling & Extensions (.ts vs .tsx)
**🔥 Scenario (Full-Stack Component Architecture):**
Internal logic utilities and caching services are strictly labeled `.ts`. Contrastingly, React UX frontend components use `.tsx` to empower TypeScript to dissect and validate embedded JSX HTML syntax elements concurrently.

### 16. "in" Operator
**🔥 Scenario (Role-Based Access Control / RBAC Validation):**
Upon consuming a JWT Auth Payload, the object profile might manifest as an `AdminNode` or `CustomerNode`. Interrogating the node utilizing `if ("adminLevel" in userProfile)` allows safe dynamic routing to sensitive dashboard areas.

### 17. Union Types
**🔥 Scenario (Payment Gateway Strategy Selection):**
```typescript
type CheckoutMethod = "credit_card" | "upi" | "wallet_balance";
let activeMethod: CheckoutMethod = "upi"; 
```
Anchoring the variable explicitly to these three static strings renders comprehensive backend switch-case validation virtually bulletproof.

### 19-22. Syntax Features
**🔥 Scenario (Sports Statistics Display - e.g., Cricbuzz):**
```typescript
const renderMatchScore = (batsman: string, totalRuns?: number): void => {
    console.log(`Current tally for ${batsman} remains ${totalRuns ? totalRuns : 'Did Not Bat'}`);
};
```

### 24. Interfaces
**🔥 Scenario (React Shared Component Library Framework):**
```typescript
interface CoreButtonProps {
    actionLabel: string;
    onInteract: () => void;
    isDisabled?: boolean;
}
```
Standardizing props dictates uniformity inside massive UI repositories, compelling junior developers to supply the precise, non-negotiable parameters for core designs.

---

## 🟡 TypeScript Intermediate Interview Questions

### 26. Never Type
**🔥 Scenario (Critical System Halt Protocols):**
```typescript
function invokeSystemFailure(errorLog: string): never {
    throw new Error(`CRITICAL KERNEL PANIC: ${errorLog}`);
}
```
Injecting `never` ensures the compiler unconditionally acknowledges code execution termination logic. It secures exhaustiveness checking mechanics frequently utilized recursively in Redux reducers.

### 27. Enums
**🔥 Scenario (Supply Chain / Amazon Tracking Engine):**
```typescript
enum DeliveryState { PendingProcessing = 1, ShippedStatus, SuccessfullyDelivered, PaymentCancelled }
```
Translating unreadable raw database enumeration integers into lucid, recognizable statuses establishes impeccable code clarity for drop-down menus, granular data analytics, and sorting filters.

### 28-29. Destructuring & Inference
**🔥 Scenario (Redux Centralized State Actions):**
Fluidly extracting `{ payload, type }` within a reducer action block. Simultaneously, TypeScript automatically infers the `payload` configuration footprint without burdening the file with heavy, repetitive annotation artifacts.

### 30-32. Modules & tsconfig.json
**🔥 Scenario (Distributed Enterprise Micro-Frontends):**
An enterprise platform partitions logic horizontally across `billing_api.ts`, `auth_layer.ts`, and `user_metrics.ts`. These domains are tightly governed by one ultimate parent `tsconfig.json` executing `strict: true` compliance, aligning code habits across disparate remote squads.

### 33. Decorators
**🔥 Scenario (NestJS Secure Routing Operations):**
```typescript
@Controller('protected/users')
class UserDataController {
    @Get()
    @UseGuards(JwtAuthGuard)
    fetchUserData() { ... }
}
```
Decorators inject profound routing architectures and layer HTTP JWT validation directly atop endpoints, abstracting massive validation middleware clutter beautifully.

### 36. Calling Base Constructors
**🔥 Scenario (Consolidated Authentication Services):**
A monolithic `CoreAuth` base class calculates foundational cryptographic token validity mapping. The subsequent child `DashboardAuth` orchestrator mandates `super(token)` sequentially within its initializing construct, absorbing global checks prior to unlocking VIP admin UI features.

### 41. Mixins
**🔥 Scenario (Modern Game Design / Engine Logic Architecture):**
Rather than wrestling with convoluted, deep, and unyielding inheritance trees, logic developers synthesize standalone `Flyable` and `Shootable` Mixin trait functions. They fuse these directly onto an agnostic `BaseVehicle` entity mapping to spawn an agile, customized Attack Helicopter object.

### 42. Immutable Object Properties
**🔥 Scenario (Critical Environment Bootstrap Variables):**
```typescript
interface InitializationConfig {
    readonly clusterEndpoint: string;
    readonly buildTargetVersion: string;
}
```
Executing immutable enforcement constructs guarantees no fragmented asynchronous service accidentally overtakes and corrupts primary environment connection strings.

---

## 🔴 TypeScript Interview Questions For Experienced

### 43-44. Classes vs Interfaces
**🔥 Scenario (Node.js Express Backend Pipeline):**
Architects wield pure interfaces exclusively at HTTP ingest protocols, validating the skeleton of approaching JSON POST footprints. Conversely, robust Classes deploy into service layers (like `PaymentGatewayService`), hosting algorithmic logic, maintaining encapsulated dependency states, and firing database migrations.

### 45-47. Visibility Methods
**🔥 Scenario (Core Encryption Protocols):**
Concealed inside a `TokenService` component, the delicate `processCryptography()` module is purposefully obscured with the `private` accessor tag. This barricades other rogue routing scripts from improperly surfacing decrypted credit hashes.

### 48. Declaration (.d.ts) Files
**🔥 Scenario (NPM Corporate Library Dispatching):**
In exporting a generic corporate structural UI NPM artifact, engineering groups manufacture bundled `.d.ts` configurations. This empowers completely separate external consumption teams to harvest autocompletion and logic validation within their IDEs, circumventing the need heavily reverse-engineer opaque trans-piled code.

### 50. Conditional Typing
**🔥 Scenario (Building Advanced Generic React Layouts):**
```typescript
type DynamicRenderProps<DataType> = DataType extends string 
    ? { textValue: DataType; onChangeEvent: (val: DataType) => void } 
    : { complexReferenceFrame: DataType; onMountLoadEvent: () => void };
```
Structuring remarkably polymorphic UI table forms. Should the overarching system pipe in primitive strings, it aggressively demands an `onChangeEvent` hook mapping. If it fields abstract array data structures, it pivots dynamically to strictly request an `onMountLoadEvent` hook mapping, imposing uncompromising architectural adherence based distinctly on payload configurations.
