# QuizPlatform

A single-page quiz app: a Vue 3 frontend that presents a multiple-choice quiz and the small Spring Boot service that serves the question bank from MongoDB. UI text and code comments are Chinese; this document is English.

Solo project on the [Seal-Re](https://github.com/Seal-Re) account. MIT licensed. The backend module keeps its original name `vidar` (`com.seal.vidar`).

## Backend

9 Java files, 151 lines. Spring Boot 2.5.0 on Java 17, `spring-boot-starter-data-mongodb`, no relational database.

```
controller/ApiController.java        GET /api/questions
service/ApiService.java              interface
service/impl/ApiServiceImpl.java     findAll() wrapped in the response DTO
repository/QuestionRepository.java   MongoRepository<QuestionDao, String>
entity/dao/QuestionDao.java          the MongoDB document
entity/dto/QuestionsDto.java         API response shape
```

| Method | Path | Response |
|---|---|---|
| `GET` | `/api/questions` | `{"questionList":[{...document...}]}` — the whole `questions` collection, unpaged |

`QuestionDao` is the document model (`@Document(collection = "questions")`): `ObjectId id`, `String question`, `String answer`, `List<String> options`. `repository.findAll()` is the only query used.

`entity/bo/Questions.java` is dead code — it is imported by `ApiService` but never referenced. `QuestionsDto` wraps `List<QuestionDao>`, so the storage model and the API payload are the same class.

Configuration (`src/main/resources/application.properties`):

```properties
server.port=4399
spring.data.mongodb.host=<mongo-host>
spring.data.mongodb.port=27017
spring.data.mongodb.database=vidar
```

## Frontend (`frontend/`)

Vue 3.5 + Vite 6 + Element Plus 2.9 + Pinia 3 (`pinia-plugin-persistedstate`) + Vue Router 4 + axios. One route, `/` → `views/Vidar/AppIndex.vue`:

- On mount, if the persisted Pinia store is empty it calls `GET /api/questions`; otherwise it reuses the stored question list.
- Quiz UI: question card with radio options, a numbered question grid (answered/current states), previous/next buttons, submit on the last question, then a score screen with a per-question review toggle.
- Answers are held in component state plus the `question_train` Pinia store (localStorage-persisted). Nothing is sent back to the server.

Notes on the current state: the fetched list is filtered with `q.model === '0'`, but the backend document has no `model` field, so the filter keeps everything; the "return home" button pushes `/home`, a route that does not exist; and the nine Pinia modules under `src/store/modules/` (`user`, `patient`, `container`, `tabs`, `breadcrumb`, `collapse`, `loading`, `questions`, `questions_train`) are only partly wired up — the view imports `questions_train`, the rest are scaffolding carried over from another application, as are the unused dependencies `echarts`, `vanta`, `three`, `sortablejs`, `vue3-markdown-it`, `lucide-react`, `date-fns` and a Tailwind configuration whose `content` globs match no files.

## Getting started

```bash
# backend — set the MongoDB coordinates in application.properties first
mvn spring-boot:run              # port 4399

# frontend
cd frontend && npm install && npm run dev
```

The frontend has no dev proxy configured, so during development it expects the API on the same origin (or a proxy/reverse proxy of your own) and a `questions` collection available in MongoDB. `src/main/resources/static/index.html` is a leftover placeholder page.

## Limitations

- One endpoint, read-only: there is no way to add questions, submit a score or store results.
- No pagination, category or filtering; the whole collection is returned.
- No tests of substance (the single generated `VidarApplicationTests` file) and no build or deploy configuration (no Dockerfile, no CI).

## License

MIT © [Seal-Re](https://github.com/Seal-Re)
