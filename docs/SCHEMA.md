# Schema Design

## 1. ASCII Visual Entity-Relationship Diagram

```text
  ┌───────────────────────────┐                        ┌───────────────────────────┐
  │           USER            │                        │       ORGANISATION        │
  ├───────────────────────────┤                        ├───────────────────────────┤
  │ PK  id                    │◄────────┐    ┌────────►│ PK  id                    │
  │     email (UQ)            │         │    │         │     name                  │
  │     username              │         │    │         │     description           │
  │     password_hash         │         │    │         │     created_at            │
  │     created_at            │         │    │         └─────────────┬─────────────┘
  └─────────────┬─────────────┘         │    │                       │
                │                       │    │                       │ 1
                │ 1              ┌──────┴────┴───────────────┐       │
                │                │    ORGANISATION_MEMBER    │       │
                │                ├───────────────────────────┤       │
                │                │ PK    id                  │       │
                │                │ FK,UQ user_id             │       │
                │                │ FK,UQ org_id              │       │
                │                │       role (OWNER/MEMBER) │       │
                │                └───────────────────────────┘       │
                │                                                    │
                │                                                    ▼ N
                │                                      ┌───────────────────────────┐
                │                                      │           BOARD           │
                │                                      ├───────────────────────────┤
                │                                      │ PK  id                    │
                │                                      │     title                 │
                │                                      │ FK  org_id                │
                │                                      │     created_at            │
                │                                      └─────────────┬─────────────┘
                │                                                    │ 1
                │                                                    │
                │                                                    ▼ N
                │                                      ┌───────────────────────────┐
                │                                      │          SECTION          │
                │                                      ├───────────────────────────┤
                │                                      │ PK  id                    │
                │                                      │     title                 │
                │                                      │     position (Float/Int)  │
                │                                      │ FK  board_id              │
                │                                      └─────────────┬─────────────┘
                │                                                    │ 1
                │                                                    │
                │                                                    ▼ N
                │                                      ┌───────────────────────────┐
                │ 1                                    │           ISSUE           │
                │                                      ├───────────────────────────┤
                │                      ┌──────────────►│ PK  id                    │
                │                      │               │     title                 │
                │                      │               │     description           │
                │                      │               │     position (Float/Int)  │
                │                      │               │     priority (LOW/MED/HI) │
                │                      │               │ FK  section_id            │
                │                      │               │     created_at            │
                │                      │               └──────┬──────────────┬─────┘
                │                      │                      │ 1            │ 1
                │       ┌──────────────┴────────────┐         │              │
                │       │      ISSUE_ASSIGNEE       │         │              │
                │       ├───────────────────────────┤         │              │
                │       │ PK    id                  │         │              │
                └──────►│ FK,UQ user_id             │         │              │
                        │ FK,UQ issue_id            │         │              │
                        └───────────────────────────┘         │              │
                                                              ▼ N            │
                                                ┌──────────────────────────┐ │
                                                │         COMMENT          │ │
                                                ├──────────────────────────┤ │
                                                │ PK  id                   │ │
                                                │     content (Text)       │ │
                                                │ FK  user_id              │◄┘
                                                │ FK  issue_id ◄───────────┘
                         ┌─────────────────────►│ FK  parent_comment_id    │ (Nullable for replies)
                         │                      │     created_at           │
                         └──────────────────────┴──────────────────────────┘ (Self Reference)
2. Interactive Mermaid ER Diagram
erDiagram
    USER ||--o{ ORGANISATION_MEMBER : has
    ORGANISATION ||--o{ ORGANISATION_MEMBER : has
    ORGANISATION ||--o{ BOARD : contains
    BOARD ||--o{ SECTION : contains
    SECTION ||--o{ ISSUE : contains
    USER ||--o{ ISSUE_ASSIGNEE : assigned_to
    ISSUE ||--o{ ISSUE_ASSIGNEE : has_assignee
    USER ||--o{ COMMENT : writes
    ISSUE ||--o{ COMMENT : has
    COMMENT ||--o{ COMMENT : replies_to

    USER {
        Long id PK
        String email UK
        String username
        String password_hash
        Timestamp created_at
    }

    ORGANISATION {
        Long id PK
        String name
        String description
        Timestamp created_at
    }

    ORGANISATION_MEMBER {
        Long id PK
        Long user_id FK
        Long org_id FK
        String role
    }

    BOARD {
        Long id PK
        String title
        Long org_id FK
        Timestamp created_at
    }

    SECTION {
        Long id PK
        String title
        Double position
        Long board_id FK
    }

    ISSUE {
        Long id PK
        String title
        String description
        Double position
        String priority
        Long section_id FK
        Timestamp created_at
    }

    ISSUE_ASSIGNEE {
        Long id PK
        Long user_id FK
        Long issue_id FK
    }

    COMMENT {
        Long id PK
        String content
        Long user_id FK
        Long issue_id FK
        Long parent_comment_id FK
        Timestamp created_at
    }
3. Entity Definitions & Field Details
1. User
- id (PK, Long / BIGSERIAL)
- email (VARCHAR, Unique)
- username (VARCHAR)
- password_hash (VARCHAR)
- created_at (TIMESTAMP)
2. Organisation
- id (PK, Long / BIGSERIAL)
- name (VARCHAR)
- description (TEXT, Nullable)
- created_at (TIMESTAMP)
3. OrganisationMember (User <-> Org)
- id (PK, Long / BIGSERIAL)
- user_id (FK -> User)
- org_id (FK -> Organisation)
- role (VARCHAR / ENUM: OWNER, ADMIN, MEMBER)
- Constraint: UNIQUE(user_id, org_id)
4. Board
- id (PK, Long / BIGSERIAL)
- title (VARCHAR)
- org_id (FK -> Organisation)
- created_at (TIMESTAMP)
5. Section (Column / List)
- id (PK, Long / BIGSERIAL)
- title (VARCHAR)
- position (DOUBLE PRECISION - for fractional drag-drop reordering)
- board_id (FK -> Board)
6. Issue (Card)
- id (PK, Long / BIGSERIAL)
- title (VARCHAR)
- description (TEXT, Nullable)
- position (DOUBLE PRECISION - for fractional drag-drop reordering)
- priority (VARCHAR / ENUM: LOW, MEDIUM, HIGH)
- section_id (FK -> Section)
- created_at (TIMESTAMP)
7. IssueAssignee (User <-> Issue)
- id (PK, Long / BIGSERIAL)
- user_id (FK -> User)
- issue_id (FK -> Issue)
- Constraint: UNIQUE(user_id, issue_id)
8. Comment
- id (PK, Long / BIGSERIAL)
- content (TEXT)
- user_id (FK -> User)
- issue_id (FK -> Issue)
- parent_comment_id (FK -> Comment, Nullable - for nested replies)
- created_at (TIMESTAMP)
