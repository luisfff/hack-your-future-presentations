# Databases Week 1 — Example Solutions

These are example answers to the exercises in the [session plan](hackyourfuture-databases-week1-session-plan.md). Other queries may also be correct. They use SQLite and the supplied [Tasks database](session-materials/tasks.sqlite3).

Use a downloaded **copy** of the database. `UPDATE`, `INSERT`, `DELETE`, and `CREATE TABLE` change it. Run the statements for one exercise at a time, not this whole document as a script. Check the rows with a `SELECT` before changing or deleting them, and commit changes in DBeaver if necessary. The homework challenges should each start with a fresh copy of the sample data, as they do in SQLPlay.

The sample status IDs are `1` for Not started, `2` for In progress, and `3` for Done. Results can differ if you have already changed your database.

## 2.4 Explore the Database

1. The tables are `user`, `task`, `status`, `tag`, and `task_tag`.
2. In DBeaver, double-click a table and open its **ER Diagram** tab to see its relationships.
3. The `task` columns are `id` (primary key), `title`, `description`, `created`, `updated`, `due_date`, `status_id`, and `user_id`. The last two link to `status` and `user`.
4. It stores tasks: their titles, descriptions, dates, status, and assigned user.
5. Yes. Double-click a table and open its **Data** tab.
6. Task `18` is **Learn how databases work**. You can also find it with:

   ```sql
   SELECT title FROM task WHERE id = 18;
   ```

## 3.7 Practice SELECT Queries

```sql
-- 1. User names and phones
SELECT name, phone FROM user;

-- 2. Users ordered by name
SELECT * FROM user ORDER BY name;

-- 3. Name of user 10
SELECT name FROM user WHERE id = 10;

-- 4. Three oldest tasks
SELECT * FROM task ORDER BY created ASC LIMIT 3;

-- 5. Tasks without a description
SELECT * FROM task WHERE description IS NULL;

-- 6. Users whose email ends in .com
SELECT * FROM user WHERE email LIKE '%.com';

-- 7. Titles of tasks that are not started
SELECT title FROM task WHERE status_id = 1;

-- 8. User 1's tasks in progress
SELECT * FROM task WHERE user_id = 1 AND status_id = 2;

-- 9. Tasks whose title contains database
SELECT * FROM task WHERE title LIKE '%database%';

-- 10. Names of users without a phone number
SELECT name FROM user WHERE phone IS NULL;

-- 11. Five newest tasks
SELECT title, created FROM task ORDER BY created DESC LIMIT 5;

-- 12. Tasks not started or in progress, ordered by title
SELECT * FROM task WHERE status_id IN (1, 2) ORDER BY title;
```

In SQLite, `LIKE` is case-insensitive for the ASCII letters in `database` and `.com` by default. `IS NULL` checks for missing values, not empty strings.

## 3.8 Practice UPDATE

Before each update, select the row using the same `WHERE` condition; select it again afterward to check the result.

```sql
-- 1. Mark task 1 Done
UPDATE task SET status_id = 3 WHERE id = 1;

-- 2. Change task 2's description
UPDATE task SET description = 'Write and test SELECT queries' WHERE id = 2;

-- 3. Change task 3's title
UPDATE task SET title = 'Practice: Review query results' WHERE id = 3;

-- 4. Change task 4's due date
UPDATE task SET due_date = '2031-01-15' WHERE id = 4;

-- 5. Change user 1's phone number
UPDATE user SET phone = '12345678' WHERE id = 1;

-- 6. Change tag 1's name
UPDATE tag SET name = 'Practice' WHERE id = 1;
```

For example, use `SELECT * FROM task WHERE id = 1;` before and after the first update.

## 3.9 Practice INSERT

Run these on a copy of the database. The example user and tag names are choices, not required values. If you have run an insert before, choose distinctive values and record the IDs of the rows you just added for section 3.10.

```sql
-- 1. Task assigned to user 1, not started
INSERT INTO task (title, created, updated, status_id, user_id)
VALUES ('Practice 1: Read SQL', datetime('now'), datetime('now'), 1, 1);
SELECT * FROM task WHERE id = last_insert_rowid();

-- 2. Task with a description, not started
INSERT INTO task (title, description, created, updated, status_id)
VALUES ('Practice 2: Write queries', 'Practice SELECT statements', datetime('now'), datetime('now'), 1);

-- 3. Task assigned to user 1, in progress
INSERT INTO task (title, created, updated, status_id, user_id)
VALUES ('Practice 3: Review results', datetime('now'), datetime('now'), 2, 1);

-- 4. Task with a due date, not started
INSERT INTO task (title, created, updated, due_date, status_id)
VALUES ('Practice 4: Plan a project', datetime('now'), datetime('now'), '2030-12-31', 1);

-- 5. User without a phone or assigned tasks
INSERT INTO user (name, email)
VALUES ('Practice Student 3.9', 'practice-student-3-9@example.com');
SELECT * FROM user WHERE id = last_insert_rowid();

-- 6. New tag
INSERT INTO tag (name) VALUES ('Practice Tag 3.9');
SELECT * FROM tag WHERE id = last_insert_rowid();
```

Run each `SELECT ... last_insert_rowid()` immediately after its corresponding insert on the same connection. Record the three IDs; SQLite assigns them automatically.

## 3.10 Practice DELETE

Use the IDs you recorded in section 3.9. Replace each `:..._id` placeholder with the actual numeric ID before running the statements. Check that the selected row is the one you added and that the user or tag has no related tasks or associations. Stop if it is not the intended row. After deletion, rerun the `SELECT` to confirm that no row remains.

```sql
-- 1. Delete the task from INSERT exercise 1
SELECT * FROM task WHERE id = :task_id;
DELETE FROM task WHERE id = :task_id;

-- 2. Delete the user from INSERT exercise 5
SELECT * FROM user WHERE id = :user_id;
SELECT * FROM task WHERE user_id = :user_id;
DELETE FROM user WHERE id = :user_id;

-- 3. Delete the tag from INSERT exercise 6
SELECT * FROM tag WHERE id = :tag_id;
SELECT * FROM task_tag WHERE tag_id = :tag_id;
DELETE FROM tag WHERE id = :tag_id;
```

## 3.11 Create a Table

This section is already a worked example in the session plan. If `practice_note` already exists, choose another name and use it in all three statements.

```sql
CREATE TABLE practice_note (
    id INTEGER PRIMARY KEY,
    title TEXT NOT NULL,
    details TEXT
);
INSERT INTO practice_note (title, details)
VALUES ('First note', 'Practising CREATE TABLE');
SELECT * FROM practice_note;
```

## 3.12 Interactive Practice

Run each step separately. After inserting the task, record the ID returned by the immediately following `SELECT` on the same connection. Replace `:task_id` with that ID; check the selected row before updating or deleting it.

```sql
SELECT * FROM task;

INSERT INTO task (title, created, updated, status_id, user_id)
VALUES ('Prepare presentation', datetime('now'), datetime('now'), 1, 1);
SELECT * FROM task WHERE id = last_insert_rowid();

SELECT * FROM task WHERE id = :task_id;
UPDATE task SET status_id = 2 WHERE id = :task_id;
SELECT * FROM task WHERE id = :task_id;

DELETE FROM task WHERE id = :task_id;
SELECT * FROM task WHERE id = :task_id;
```

## 3.13 Combine Everything

This example uses a made-up user and book. Choose your own name and email if you prefer. Run the steps in order, and record the generated IDs immediately after each insert on the same connection. Replace `:user_id`, `:first_task_id`, and `:second_task_id` with those numeric IDs in the later statements. Before updating or deleting, check that the selected row is the intended task.

```sql
-- 1 and 2. Add a user and find their ID
INSERT INTO user (name, email)
VALUES ('Practice Student 3.13', 'practice-student-3-13@example.com');
SELECT * FROM user WHERE id = last_insert_rowid();

-- 3. Create two tasks for that user and record their IDs
INSERT INTO task (title, description, created, updated, status_id, user_id)
VALUES ('Read about SQL', 'Read the session notes', datetime('now'), datetime('now'), 1, :user_id);
SELECT * FROM task WHERE id = last_insert_rowid();

INSERT INTO task (title, description, created, updated, status_id, user_id)
VALUES ('Practice SQL', 'Write some queries', datetime('now'), datetime('now'), 1, :user_id);
SELECT * FROM task WHERE id = last_insert_rowid();

-- 4. Mark the first task in progress
SELECT * FROM task WHERE id = :first_task_id AND user_id = :user_id;
UPDATE task SET status_id = 2 WHERE id = :first_task_id AND user_id = :user_id;
SELECT * FROM task WHERE id = :first_task_id AND user_id = :user_id;

-- 5. Show the user's task titles and descriptions
SELECT title, description FROM task WHERE user_id = :user_id;

-- 6. Delete the second task after checking it
SELECT * FROM task WHERE id = :second_task_id AND user_id = :user_id;
DELETE FROM task WHERE id = :second_task_id AND user_id = :user_id;
SELECT * FROM task WHERE id = :second_task_id AND user_id = :user_id;

-- 7 and 8. Create a table, insert a book, and inspect it
CREATE TABLE practice_book (
    id INTEGER PRIMARY KEY,
    title TEXT NOT NULL,
    author TEXT
);
INSERT INTO practice_book (title, author) VALUES ('Learning SQL', 'Example Author');
SELECT id, title, author FROM practice_book;
```

If `practice_book` already exists, choose a different table name and use it for the `CREATE`, `INSERT`, and `SELECT`. Refresh the Database Navigator to see it.

## Homework Questions and Challenges

Each answer is independent: use the fresh sample database provided by each SQLPlay challenge, or restore a fresh copy before trying another answer locally. In particular, running challenges 14–19 sequentially changes data needed by later challenges.

```sql
-- 1. Count the tasks
SELECT COUNT(*) FROM task;

-- 2. Count missing or unparseable due dates
SELECT COUNT(*) FROM task WHERE datetime(due_date) IS NULL;

-- 3. Done tasks
SELECT * FROM task WHERE status_id = 3;

-- 4. Tasks not done
SELECT * FROM task WHERE status_id != 3;

-- 5. Tasks by creation date, newest first
SELECT * FROM task ORDER BY created DESC;

-- 6. Most recently created task
SELECT * FROM task ORDER BY created DESC LIMIT 1;

-- 7. Tasks mentioning database in the title or description
SELECT title, due_date FROM task
WHERE LOWER(title) LIKE '%database%' OR LOWER(description) LIKE '%database%';

-- 8. Task titles and IDs
SELECT id, title FROM task ORDER BY id ASC;

-- 9. Tasks in progress
SELECT id, title FROM task WHERE status_id = 2 ORDER BY id ASC;

-- 10. Tasks without due dates
SELECT id, title FROM task WHERE due_date IS NULL ORDER BY id ASC;

-- 11. Tasks alphabetically
SELECT id, title FROM task ORDER BY title ASC;

-- 12. Five highest task IDs
SELECT id, title FROM task ORDER BY id DESC LIMIT 5;

-- 13. Unassigned tasks
SELECT id, title FROM task WHERE user_id IS NULL ORDER BY id ASC;

-- 14. Rename task 1
UPDATE task SET title = 'Wash the laundry' WHERE id = 1;

-- 15. Mark task 22 done
UPDATE task SET status_id = 3 WHERE id = 22;

-- 16. Assign task 23 to user 11
UPDATE task SET user_id = 11 WHERE id = 23;

-- 17. Set the status of the two unassigned sample tasks
UPDATE task SET status_id = 1 WHERE id IN (23, 35) AND user_id IS NULL;

-- 18. Delete task 35 only
DELETE FROM task WHERE id = 35;

-- 19. Remove task 1's Personal tag association only
DELETE FROM task_tag WHERE task_id = 1 AND tag_id = 2;

-- 20. Create task notes
CREATE TABLE task_note (
    id INTEGER PRIMARY KEY,
    task_id INTEGER NOT NULL,
    note TEXT NOT NULL,
    FOREIGN KEY (task_id) REFERENCES task(id)
);
```
