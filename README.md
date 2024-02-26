## SQL CRUD

### the SQL code to create each of the required tables

#### Part 1:

```sqlite
CREATE TABLE restaurants (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    category TEXT,
    price_tier TEXT,
    neighborhood TEXT,
    opening_hours TEXT,
    average_rating REAL,
    good_for_kids BOOLEAN
);
CREATE TABLE reviews (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    restaurant_id INTEGER,
    user_name TEXT NOT NULL,
    rating INTEGER CHECK(rating >= 1 AND rating <= 5),
    comment TEXT,
    created_at DATETIME DEFAULT (datetime('now','localtime')),
    FOREIGN KEY (restaurant_id) REFERENCES restaurants(id)
);
```

#### Part 2:

```sqlite
CREATE TABLE users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    email TEXT UNIQUE NOT NULL,
    password TEXT NOT NULL,
    handle TEXT UNIQUE NOT NULL
);

CREATE TABLE posts (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER,
    post_type TEXT CHECK(post_type IN ('message', 'story')),
    content TEXT NOT NULL,
    recipient_id INTEGER,
    isView BOOLEAN,
    created_at DATETIME DEFAULT (datetime('now','localtime')),
    FOREIGN KEY (user_id) REFERENCES users(id),
    FOREIGN KEY (recipient_id) REFERENCES users(id)
);
```

### a link to each of the practice CSV data files in the `data` directory.

[restaurants.csv](./data/restaurants.csv)

[users.csv](./data/users.csv)

[posts.csv](./data/posts.csv)

### the SQLite code to import the practice CSV data files into the tables.

#### Part 1:

sqlite3 database.db ".mode csv" ".import restaurants.csv restaurants"
Create a database named database and import the data from restaurants.csv into a table named restaurants.

#### Part 2:

sqlite3 database2.db ".mode csv" ".import users.csv users"
Create a database named database2 and import the data from users.csv into a table named users.

sqlite3 database2.db ".mode csv" ".import posts.csv posts"
Import the data from posts.csv into the table named posts.

### the SQL queries that solve each of the tasks you were asked to do. Make it clear which task each query is intended to solve - include the task number and text on the line above the SQL code solution.

#### Part 1 Queries:

1. Find all cheap restaurants in a particular neighborhood (pick any neighborhood as an example).

   ```sqlite
   SELECT * FROM restaurants WHERE neighborhood = 'Bayside' AND price_tier = 'cheap';
   ```

2. Find all restaurants in a particular genre (pick any genre as an example) with 3 stars or more, ordered by the number of stars in descending order.

   ```sqlite
   SELECT * FROM restaurants WHERE category = 'Lebanese' AND average_rating >= 3 ORDER BY average_rating DESC;
   ```

3. Find all restaurants that are open now (see hint below).

   ```sqlite
   SELECT * FROM restaurants WHERE strftime('%H:%M', 'now', 'localtime') BETWEEN SUBSTR(opening_hours, 1, 5) AND SUBSTR(opening_hours, 9, 5);
   ```

4. Leave a review for a restaurant (pick any restaurant as an example; note that leaving a review has no automatic effect on the average rating of the restaurant).

   ```sqlite
   INSERT INTO reviews (restaurant_id, user_name, rating, comment) VALUES (5, 'Pub', 3, 'The food was rather average, not good or bad');
   ```

5. Delete all restaurants that are not good for kids.

   ```sqlite
   DELETE FROM restaurants WHERE good_for_kids = 'FALSE';
   ```

6. Find the number of restaurants in each NYC neighborhood.

   ```sqlite
   SELECT COUNT(*) AS restaurant_count FROM restaurants GROUP BY neighborhood;
   ```

#### Part 2 Queries:

1. Register a new User.

   ```sqlite
   INSERT INTO users (email, password, handle) VALUES ('newuser@example.com', 'password123', 'newuserhandle');
   ```

2. Create a new Message sent by a particular User to a particular User (pick any two Users for example).

   ```sqlite
   INSERT INTO posts (user_id, post_type, content, recipient_id, isView) VALUES (1, 'message', 'Hey, how are you doing today?', 2, 'FALSE');
   ```

3. Create a new Story by a particular User (pick any User for example).

   ```sqlite
   INSERT INTO posts (user_id, post_type, content, recipient_id, isView) VALUES (1, 'story', 'Exploring the new coffee shop in town!', NULL, 'FALSE');
   ```

4. Show the 10 most recent visible Messages and Stories, in order of recency.

   ```sqlite
   SELECT *
   FROM posts
   WHERE 
       (post_type = 'message' AND isView = 'FALSE')
       OR
       (post_type = 'story' AND isView = 'TRUE')
   ORDER BY created_at DESC
   LIMIT 10;
   ```

5. Show the 10 most recent visible Messages sent by a particular User to a particular User (pick any two Users for example), in order of recency.

   ```sqlite
   SELECT *
   FROM posts
   WHERE user_id = 1 AND recipient_id = 2 AND post_type = 'message' AND isView = 'FALSE'
   ORDER BY created_at DESC
   LIMIT 10;
   ```

6. Make all Stories that are more than 24 hours old invisible.

   ```sqlite
   UPDATE posts
   SET isView = 'FALSE'
   WHERE post_type = 'story' AND created_at <= datetime('now', '-1 day', 'localtime');
   ```

7. Show all invisible Messages and Stories, in order of recency.

   ```sqlite
   SELECT *
   FROM posts
   WHERE 
       (post_type = 'message' AND isView = 'TRUE')
       OR
       (post_type = 'story' AND isView = 'FALSE')
   ORDER BY created_at DESC;
   ```

8. Show the number of posts by each User.

   ```sqlite
   SELECT user_id, COUNT(*) AS post_count
   FROM posts
   GROUP BY user_id;
   ```

9. Show the post text and email address of all posts and the User who made them within the last 24 hours.

   ```sqlite
   SELECT p.content AS post_text, u.email
   FROM posts p
   JOIN users u ON p.user_id = u.id
   WHERE p.created_at >= datetime('now', '-1 day', 'localtime');
   ```

10. Show the email addresses of all Users who have not posted anything yet.

    ```sqlite
    SELECT u.email
    FROM users u
    LEFT JOIN posts p ON u.id = p.user_id
    WHERE p.id IS NULL;
    ```