# Node.js Interview Questions and Answers

The answers below are written to be useful in an interview: start with the concise explanation, then add detail or an example when the interviewer asks a follow-up.

## 1. What is Node.js?

Node.js is an open-source, cross-platform JavaScript runtime that runs JavaScript outside a web browser. It is built on Chrome's V8 JavaScript engine and adds APIs for tasks such as working with files, networking, streams, and operating-system resources.

Node.js is commonly used for web APIs, real-time applications, command-line tools, and other I/O-heavy services. It is a runtime, not a programming language or a web framework. Frameworks such as Express and Fastify run on Node.js.

## 2. Compare Java and Node.js

Java is a programming language and runtime ecosystem; Node.js is a JavaScript runtime. A fair comparison is usually between a Java application and a Node.js application.

| Area | Java | Node.js |
|---|---|---|
| Language and runtime | Usually Java on the JVM | JavaScript (or compiled-to-JavaScript languages) on V8 |
| Typical concurrency model | Commonly uses multiple threads and thread pools | JavaScript callbacks normally run on one event-loop thread; asynchronous I/O and runtime workers handle work around it |
| I/O | Often synchronous or asynchronous, depending on APIs and design | Asynchronous, non-blocking I/O is a common default pattern |
| Strengths | Mature ecosystem, static typing, strong tooling, and good support for large, CPU-intensive or enterprise systems | Fast development, one language across many web stacks, and efficient handling of many concurrent I/O operations |
| Trade-offs | Threading and runtime configuration can add complexity and resource overhead | CPU-heavy JavaScript can block the event loop; type safety depends on the language and tooling chosen |

Neither is universally faster or better. The right choice depends on workload, team expertise, ecosystem, and operational requirements.

## 3. What do single-threaded and multithreaded mean?

A **thread** is an execution path within a process. In a single-threaded model, one thread executes a sequence of instructions at a time. In a multithreaded model, a process uses multiple threads that can make progress concurrently; on multiple CPU cores, they may run in parallel.

In Node.js, JavaScript callbacks in a given isolate generally execute on one main thread. Node can still perform asynchronous I/O, and its runtime uses other threads for certain tasks. Applications can also use `worker_threads` to run JavaScript in parallel. So “Node.js is single-threaded” is a useful shorthand for its usual JavaScript execution model, not a claim that the entire runtime has only one thread.

## 4. What is libuv?

libuv is a C library used by Node.js to provide its event loop and cross-platform asynchronous I/O abstractions. It helps Node.js work consistently across operating systems.

Network I/O is generally handled asynchronously by the operating system. Some operations that do not have suitable non-blocking operating-system APIs—such as many filesystem operations and some DNS or cryptographic tasks—may be delegated to libuv's worker pool. The worker pool is distinct from the JavaScript event-loop thread.

## 5. How does memory management happen in Node.js?

V8 manages memory for JavaScript objects. It allocates objects on the heap and reclaims memory for objects that are no longer reachable using garbage collection. JavaScript developers usually do not manually allocate and free memory, but they must avoid retaining references unintentionally.

Node.js also uses memory outside the JavaScript heap—for example, buffers and native allocations. `process.memoryUsage()` can help inspect memory use. Common causes of memory growth include unbounded caches, listeners that are never removed, retained closures, and accumulating data instead of streaming it.

## 6. What is the mark-and-sweep algorithm?

Mark-and-sweep is a garbage-collection technique:

1. **Mark:** Start from roots such as global objects and active stack references, then mark every object reachable from them.
2. **Sweep:** Reclaim memory occupied by objects that were not marked.

Because unreachable objects can form cycles, this approach can collect cycles that simple reference counting would not reclaim. V8 uses a generational, optimized garbage collector, so mark-and-sweep is a useful conceptual explanation rather than a complete description of all V8 collection behavior.

## 7. What are the core capabilities or features of Node.js?

- Runs JavaScript outside the browser using V8.
- Uses an event loop and supports asynchronous, non-blocking I/O.
- Provides built-in modules for HTTP, files, streams, cryptography, operating-system information, and more.
- Supports a large package ecosystem through npm.
- Runs on major operating systems.
- Supports streaming data, which can reduce memory use for large files or responses.
- Enables real-time applications using persistent connections such as WebSockets.
- Provides worker threads and child processes for workloads that need parallelism or process isolation.

## 8. How does npm work with Node.js?

npm is the package manager most commonly used with Node.js. It is distributed with standard Node.js installers and can be used to find, install, update, and remove packages, as well as run project scripts.

When a project declares dependencies in `package.json`, `npm install` resolves and installs them into `node_modules`. The lockfile records the resolved dependency tree. For a reproducible install in automated builds, `npm ci` installs from the lockfile and fails if it does not match the manifest.

## 9. What is the default port number of Node.js?

Node.js has **no default port** for an application server. The application chooses a port when it calls methods such as `server.listen()`. Port `3000` is a common development convention, not a Node.js default. Ports `80` and `443` are conventional HTTP and HTTPS ports, respectively.

For example:

```js
const port = process.env.PORT || 3000;
server.listen(port);
```

## 10. List important npm packages for Node.js

The right packages depend on the application. Examples of commonly used packages include:

- **Web servers and routing:** Express, Fastify, Koa
- **Validation:** Zod, Joi
- **Authentication and security:** `jsonwebtoken` (JWT handling), Helmet (HTTP security headers), `bcrypt` (password hashing)
- **Database access:** `pg` (PostgreSQL), Mongoose (MongoDB), Prisma (ORM)
- **Testing:** Jest, Vitest, Supertest
- **Logging:** Pino, Winston
- **Configuration:** dotenv
- **HTTP clients:** Axios

Packages should be selected based on project needs, maintenance, security, and compatibility. Node's built-in modules can be sufficient for many tasks, so an external dependency is not always needed.

## 11. How do you validate, authenticate, and authorize applications in Node.js?

These are different responsibilities:

- **Validation** checks that input has the expected shape and values. Validate at the system boundary—for example, request parameters, body, and query string—using a schema library such as Zod or Joi. Reject invalid input and avoid trusting client-side validation alone.
- **Authentication** verifies who the caller is. Common approaches include server-side sessions with secure cookies or signed access tokens. Passwords should be stored using a password-hashing algorithm such as bcrypt or Argon2, never as plaintext or with a general-purpose fast hash.
- **Authorization** checks what an authenticated caller may do. Enforce least privilege and check permissions on the server for each protected resource or action; authentication alone does not grant access.

Also use HTTPS, secure cookie settings where applicable, rate limits for sensitive endpoints, careful secret management, and maintained dependencies. The exact design depends on the threat model and application.

## 12. What are Node.js versions?

Node.js releases use version numbers in a major, minor, and patch format, such as `22.5.1`. Major versions can introduce breaking changes; minor and patch releases generally add compatible features and fixes, respectively.

Release lines move through lifecycle stages such as Current, Active LTS, Maintenance LTS, and End-of-Life. For production, choose a currently supported LTS release unless a project has a specific compatibility requirement. The set of supported versions changes over time, so consult the official Node.js release schedule rather than relying on a memorized list.

## 13. How do you test or check Node.js versions?

Check the active versions in a terminal:

```sh
node --version
npm --version
```

In code, use `process.version` to read the Node.js version. To test an application against multiple versions, use a version manager such as `nvm` (or a platform-appropriate equivalent), install the versions required by the project, and run the test suite under each one. CI can use a version matrix for the same purpose. Use the project's `engines` field to communicate supported Node.js versions.

## 14. What are the advantages and disadvantages of Node.js?

**Advantages**

- Efficient for applications that handle many concurrent I/O operations.
- JavaScript can be used across frontend and backend teams.
- Large package ecosystem and a relatively quick development feedback loop.
- Built-in streams support processing data without loading everything into memory.
- Useful for APIs, real-time services, and command-line tools.

**Disadvantages**

- CPU-intensive work on the main JavaScript thread can block the event loop and delay unrelated requests.
- Asynchronous code can become difficult to follow without good structure and error handling.
- The large package ecosystem requires dependency review, updates, and security maintenance.
- JavaScript is dynamically typed by default; teams may need TypeScript and checks to catch errors earlier.
- Not every workload benefits from Node.js; CPU-heavy applications may need worker threads, native code, a separate service, or another runtime.

## 15. What is `npm i` or `npm install`?

`npm i` is the short alias for `npm install`.

- `npm install` with no package name installs the project's dependencies from `package.json`.
- `npm install <package>` installs a package and, by default, records it as a dependency.
- `npm install --save-dev <package>` records a development-only dependency.

For repeatable clean installs in CI, `npm ci` is often preferred when a lockfile is present and committed.

## 16. What are the advantages and disadvantages of npm?

**Advantages**

- Makes it straightforward to discover, install, and update reusable packages.
- Tracks project dependencies and scripts in `package.json`.
- Uses a lockfile to record resolved dependency versions.
- Supports development dependencies and automated installation in CI.
- Provides access to a large ecosystem of open-source packages.

**Disadvantages**

- A project can inherit vulnerabilities or maintenance problems from direct and transitive dependencies.
- Large dependency trees can increase install time, package size, and upgrade complexity.
- Version ranges without a committed lockfile can resolve to different dependency trees over time.
- Package quality and API compatibility vary, so dependencies need evaluation and maintenance.

## 17. What is `package.json`?

`package.json` is a project's npm manifest. It commonly contains the package name and version, scripts, dependency declarations, development dependencies, and metadata such as the required Node.js version.

Example:

```json
{
  "name": "example-api",
  "scripts": {
    "start": "node server.js",
    "test": "node --test"
  },
  "dependencies": {
    "express": "^5.0.0"
  },
  "engines": {
    "node": ">=22"
  }
}
```

The dependency entries usually describe acceptable version ranges; they do not necessarily pin the exact versions installed.

## 18. What is `package-lock.json`?

`package-lock.json` is npm's lockfile. It records the resolved versions and dependency-tree details for the project's installed dependencies. It helps npm reproduce an install across developer machines and CI environments.

For an application, commit the lockfile along with `package.json`. Avoid manually editing it; let npm update it when dependencies change.

## 19. What is the difference between `package.json` and `package-lock.json`?

| `package.json` | `package-lock.json` |
|---|---|
| Declares project metadata, scripts, and dependency version ranges | Records the resolved dependency tree and exact package versions |
| Usually maintained as part of project configuration | Usually generated and updated by npm |
| Describes what the project requests | Describes what npm resolved for installation |
| Required to describe an npm project | Should normally be committed for an application to make installs reproducible |

They complement each other: `package.json` expresses dependency intent, while the lockfile captures a specific resolution of that intent.

## 20. Why do we download and install npm?

npm is normally included with the official Node.js installer, so a separate installation is often unnecessary. It provides the tools to install project dependencies, run scripts, and manage packages from the npm registry. Check availability with `npm --version`.

If npm is missing or the installed version is unsuitable, install or update Node.js and npm using the method recommended for the operating system or organization. Prefer a version manager when you need to switch between Node.js versions.

## 21. Why is Node.js described as single-threaded?

Node.js is described as single-threaded because JavaScript application code usually runs on one main event-loop thread per isolate. This makes the common programming model simpler than manually coordinating many JavaScript threads.

When the application starts an asynchronous I/O operation, Node.js can delegate it to the operating system or, for some operations, libuv's worker pool. Once the operation completes, its callback or promise continuation is queued for the event loop to run. This lets one JavaScript thread make progress on other work while waiting for I/O.

This does **not** mean CPU-heavy JavaScript runs in the background: it can block the event loop. For CPU-bound parallel work, use `worker_threads`, child processes, or a separate service as appropriate.