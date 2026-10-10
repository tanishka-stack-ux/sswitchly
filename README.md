# Switchly API

A feature flag management API built with Spring Boot. You can create an organization, add projects to it, add feature flags to a project, and turn those flags on or off.

All data is stored in memory for now, so everything is lost when the app stops. There is no database yet.

## Tech stack

Java 21
Spring Boot (Spring Web, Validation)
Maven

## How to run

From inside the `api` folder:

```bash
./mvnw spring-boot:run        # macOS / Linux / Git Bash
mvnw.cmd spring-boot:run      # Windows
```

Wait for `Tomcat started on port 8080`. The API is then available at `http://localhost:8080`.

If you see `release version 21 not supported`, the terminal is using an older Java. Point `JAVA_HOME` at a JDK 21 or newer, then run the command again:

```bash
export JAVA_HOME="/path/to/your/jdk"
export PATH="$JAVA_HOME/bin:$PATH"
```

If port 8080 is already in use, stop the other copy of the app, or add `server.port: 8081` to `src/main/resources/application.yaml`.

## Project structure

```
api/src/main/java/live/switchly/api/
├── controller/    HTTP layer: reads requests, returns responses
├── service/       Business rules
├── repository/    Storage (in-memory maps for now)
├── model/         Organization, Project, Flag
├── dto/           Request bodies
└── exception/     Errors and the global error handler
```

A request always travels controller, then service, then repository. A controller only talks to a service, and a service only talks to its own repository and to other services.

## API endpoints

| Method | URL | What it does |
| ------ | --- | ------------ |
| `POST` | `/api/v1/orgs` | Create an organization |
| `GET` | `/api/v1/orgs` | List organizations |
| `GET` | `/api/v1/orgs/{orgId}` | Get one organization |
| `POST` | `/api/v1/orgs/{orgId}/projects` | Create a project in an organization |
| `GET` | `/api/v1/orgs/{orgId}/projects` | List an organization's projects |
| `GET` | `/api/v1/projects/{projectId}` | Get one project |
| `POST` | `/api/v1/projects/{projectId}/flags` | Create a flag in a project |
| `GET` | `/api/v1/projects/{projectId}/flags` | List a project's flags |
| `GET` | `/api/v1/flags/{flagId}` | Get one flag |
| `PUT` | `/api/v1/flags/{flagId}/state` | Turn a flag on or off |
| `DELETE` | `/api/v1/flags/{flagId}` | Delete a flag |

## Flag rules

A new flag always starts **off**.
  `key` is required and may only contain lowercase letters, numbers and hyphens (for example `new-checkout`).
`name` is required.
 `description` is optional.
 Two flags in the same project cannot share a key.

## Example flow

```bash
# 1. Create an organization
curl -X POST http://localhost:8080/api/v1/orgs \
  -H "Content-Type: application/json" -d '{"name": "Zomato"}'

# 2. Create a project in it (use the id from step 1)
curl -X POST http://localhost:8080/api/v1/orgs/ORG_ID/projects \
  -H "Content-Type: application/json" -d '{"name": "Consumer App"}'

# 3. Create a flag in the project (use the id from step 2)
curl -X POST http://localhost:8080/api/v1/projects/PROJECT_ID/flags \
  -H "Content-Type: application/json" \
  -d '{"key": "new-checkout", "name": "New Checkout", "description": "Redesigned checkout page"}'

# 4. Turn the flag on (use the id from step 3)
curl -X PUT http://localhost:8080/api/v1/flags/FLAG_ID/state \
  -H "Content-Type: application/json" -d '{"enabled": true}'

# 5. Check the flag
curl http://localhost:8080/api/v1/flags/FLAG_ID

# 6. Delete the flag
curl -i -X DELETE http://localhost:8080/api/v1/flags/FLAG_ID
```

## Error format

Every error uses the same shape:

```json
{"error": {"code": "NOT_FOUND", "message": "Flag 0000... not found"}}
```

Status ,Code , When
`400` , `VALIDATION_FAILED` , Missing or invalid input, such as an empty name or a bad key ,
`404` ,`NOT_FOUND` ,The organization, project or flag does not exist 
`409` ,`CONFLICT` , A flag with the same key already exists in the project 

## What I tested
Creating an organization, a project and a flag returns `201`, and the flag starts with `"enabled": false`.
Turning a flag on returns `200`, and fetching it again confirms it stayed on.
 A duplicate key in the same project returns `409`.
A key like `New Checkout` returns `400`.
An unknown id returns `404`.
Deleting a flag returns `204`, and deleting it again returns `404`.
After restarting the app, the list of organizations is empty.

##Next steps

Replace the in-memory repositories with a database. The services and controllers will not need to change, because they only depend on the repository interfaces.
