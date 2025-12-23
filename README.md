````markdown
# CSE334 Database Lab – BOOK_E Backend

Backend API for the **BOOK_E** project used in the **CSE334 Database Lab** course.

This is a Node.js + Express REST API backed by a **MySQL** database using **Sequelize** ORM.  
It supports user authentication, book listing and management, ordering between users, ratings, reviews, and genres.

---

## Main Features

-  **User authentication**
  - Registration & login with hashed passwords (bcrypt)
  - JWT-based authentication
  - User profile fetch & update (with profile picture upload)

-  **Book management**
  - Create book listings (with image upload)
  - Search and filter books
  - List all books / books by user
  - Update & delete listings
  - See uploader profile for a specific book

-  **Order system**
  - Place orders on books
  - View placed and received orders
  - Confirm / discard orders

-  **Ratings & Reviews**
  - Leave ratings for users
  - Leave written reviews
  - Fetch ratings (with average) and reviews for a user

-  **Genres**
  - Add genres
  - Retrieve all genres

-  **File uploads**
  - Static hosting for uploaded files (book images, profile pictures) via `/uploads`

---

## 🛠 Tech Stack

- **Runtime:** Node.js
- **Framework:** Express.js
- **Database:** MySQL
- **ORM:** Sequelize
- **Auth:** JSON Web Tokens (JWT), bcryptjs
- **Uploads:** multer, static `/uploads` directory
- **Other libs:**
  - `cors`
  - `body-parser`
  - `express-validator` (used in controllers)

See [`package.json`](./package.json) for full dependency list.

---

##  Project Structure

```bash
DatabaseLab_BOOK_E_Backend/
├── controllers/
│   ├── authController.js      # Login/register logic
│   ├── bookController.js      # Book CRUD, search, filter, uploader profile
│   ├── forumController.js     # (Not currently wired into routes)
│   ├── genreController.js     # Genre add & list
│   ├── orderController.js     # Place/confirm/discard orders
│   ├── ratingController.js    # Add rating & get ratings
│   ├── reviewController.js    # Add review & get reviews
│   └── userController.js      # Profile fetch & update
│
├── middleware/
│   ├── authmiddleware.js      # JWT auth check
│   └── upload.js              # Multer config for uploads
│
├── models/
│   ├── Book.js                # Book model
│   ├── Genre.js               # Genre model
│   ├── Order.js               # Order model
│   ├── Post.js                # Forum post model (unused in routes)
│   ├── Rating.js              # Rating model
│   ├── Review.js              # Review model
│   ├── User.js                # User model
│   └── database.js            # Sequelize instance, DB config & associations
│
├── routes/
│   ├── authRoutes.js          # /api/auth/...
│   ├── bookRoutes.js          # /api/book/...
│   ├── genreRoutes.js         # /api/genre/...
│   ├── orderRoutes.js         # /api/order/...
│   ├── ratingRoutes.js        # /api/rating/...
│   ├── reviewRoutes.js        # /api/review/...
│   └── userRoutes.js          # /api/user/...
│
├── uploads/                   # (created at runtime) Uploaded files
├── index.js                   # Express app entry point & route mounting
├── package.json
└── .gitignore
````

---

##  Setup & Installation

###  Prerequisites

* [Node.js](https://nodejs.org/) (recommended: v16+)
* [MySQL](https://www.mysql.com/) server
* A database named (by default) `booke`
  (You can change this in `models/database.js`.)

###  Clone the repository

```bash
git clone https://github.com/Fahad-Bin-Mahbub/DatabaseLab_BOOK_E_Backend.git
cd DatabaseLab_BOOK_E_Backend
```

###  Configure the database

Open `models/database.js` and set your MySQL credentials:

```js
const database = 'booke';
const username = 'root';
const password = 'your_mysql_password';
const host = 'localhost';
const dialect = 'mysql';

const sequelize = new Sequelize(database, username, password, {
  host: host,
  dialect: dialect,
  operatorsAliases: false,
});
```

> Make sure the `booke` database exists in MySQL (create it manually if needed).
> By default, Sequelize does `sequelize.sync({ force: false })` and will create/update tables automatically.


###  Install dependencies

```bash
npm install
```

---

## Running the Server

### Development

```bash
npm run dev
```

This uses `nodemon` to reload on file changes.

### Production / Simple run

```bash
npm start
# or
node index.js
```

By default, the server listens on:

```text
http://localhost:5000
```

You can override the port with the `PORT` environment variable:

```bash
PORT=4000 npm start
```

---

## Express App & Routing

In `index.js`, the routes are mounted as:

```js
app.use('/uploads', express.static(path.join(__dirname, 'uploads')));

app.use('/api/auth', require('./routes/authRoutes'));
app.use('/api/book', require('./routes/bookRoutes'));
app.use('/api/user', require('./routes/userRoutes'));
app.use('/api/order', require('./routes/orderRoutes'));
app.use('/api/review', require('./routes/reviewRoutes'));
app.use('/api/rating', require('./routes/ratingRoutes'));
app.use('/api/genre', require('./routes/genreRoutes'));
```

So the full API paths are `/api/<module>/...`.

---

## API Reference (Overview)

Below is a summary of the main endpoints. For a full understanding, see the corresponding controllers and routes.

### 1. Auth (`/api/auth`)

**File:** `routes/authRoutes.js` → `controllers/authController.js`

* `POST /api/auth/register`
  Register a new user.

  **Body (JSON example):**

  ```json
  {
    "username": "fahad",
    "email": "fahad@example.com",
    "password": "secret123",
    "confirmPassword": "secret123"
  }
  ```

* `POST /api/auth/login`
  Log in a user and receive a JWT token.

  **Body:**

  ```json
  {
    "email": "fahad@example.com",
    "password": "secret123"
  }
  ```

>  The same register/login handlers are also exposed under `/api/user` (see below), but `/api/auth` is the canonical auth path.

---

### 2. Users (`/api/user`)

**File:** `routes/userRoutes.js`
Controllers: `authController.js`, `userController.js`

* `POST /api/user/register`

* `POST /api/user/login`
  (Same behavior as the `/api/auth` endpoints.)

* `GET /api/user/profile`
  Get the profile of the currently authenticated user.

  * **Headers:** `Authorization: Bearer <token>`

* `GET /api/user/uploader-profile/:user_id`
  Get public profile information for a user by ID (e.g. book uploader).

* `PUT /api/user/profile/edit`
  Update the profile of the authenticated user, including profile picture.

  * **Headers:** `Authorization: Bearer <token>`
  * **Form-data:**

    * `profile_picture` – file (handled by multer in `upload.js`)
    * Other profile fields as defined in `userController.editUserProfile`

---

### 3. Books (`/api/book`)

**File:** `routes/bookRoutes.js` → `controllers/bookController.js`

* `POST /api/book/add`
  Add a new book listing.

  * **Headers:** `Authorization: Bearer <token>`
  * **Form-data:**

    * `book_img_url` – file (book image)
    * Other fields (based on `Book` model and controller), e.g.:

      ```text
      title, author, genre, price, book_condition, description, is_for_sale, is_for_exchange, ...
      ```

* `GET /api/book/search`
  Search books based on query parameters (e.g. title, author, etc.).

* `GET /api/book/filter`
  Filter books by various criteria (e.g. genre, price range, condition).

* `GET /api/book/allbooks`
  Get all available books (excluding those already transacted).

* `GET /api/book/book/:book_id`
  Get detailed information for a single book.

* `GET /api/book/userbooks/:user_id`
  Get all books uploaded by a specific user.

* `PUT /api/book/:id`
  Update book information.

* `DELETE /api/book/:id`
  Delete a book.

* `GET /api/book/uploader-profile/:book_id`
  Get uploader profile info for a given book.

---

### 4. Orders (`/api/order`)

**File:** `routes/orderRoutes.js` → `controllers/orderController.js`

All endpoints require authentication (`authmiddleware`).

* `POST /api/order/place-order`
  Place an order for a book.

  **Typical body fields:** (see `orderController.placeOrder`)

  ```json
  {
    "book_id": 1,
    "seller_id": 2,
    "shipping_address": "Some address",
    "note": "Please deliver in the evening"
  }
  ```

* `GET /api/order/user-orders`
  Get orders placed by the authenticated user (as buyer).

* `GET /api/order/received-orders`
  Get orders received by the authenticated user (as seller).

* `POST /api/order/confirm-order/:order_id`
  Confirm an order (seller action).

* `POST /api/order/discard-order/:order_id`
  Discard/cancel an order.

---

### 5. Reviews (`/api/review`)

**File:** `routes/reviewRoutes.js` → `controllers/reviewController.js`

* `POST /api/review/add-review/:user_id`
  Add a review *for* the user with ID `user_id`.

  * **Headers:** `Authorization: Bearer <token>`
  * **Body example:**

    ```json
    {
      "comment": "Very friendly and fast response!"
    }
    ```

* `GET /api/review/reviews/:user_id`
  Get all reviews for a recipient user.

---

### 6. Ratings (`/api/rating`)

**File:** `routes/ratingRoutes.js` → `controllers/ratingController.js`

* `POST /api/rating/add-rating/:user_id`
  Add a rating for the user with ID `user_id`.

  * **Headers:** `Authorization: Bearer <token>`
  * **Body example:**

    ```json
    {
      "rating": 5
    }
    ```

* `GET /api/rating/ratings/:user_id`
  Get ratings and average rating for a user.

---

### 7. Genres (`/api/genre`)

**File:** `routes/genreRoutes.js` → `controllers/genreController.js`

* `POST /api/genre/add`
  Add a new genre.

  **Body example:**

  ```json
  {
    "name": "Science Fiction"
  }
  ```

* `GET /api/genre/all`
  Get all genres.

---

##  Data Models (High-Level)

Models are defined with Sequelize in `models/` and wired together in `models/database.js`. Roughly:

* **User**

  * Basic identity (username, email, password hash)
  * Profile fields (address, profile picture, etc.)
  * Relations:

    * `User` ↔ `Book` (hasMany)
    * `User` ↔ `Order` (as buyer and seller)
    * `User` ↔ `Rating` (as rater and recipient)
    * `User` ↔ `Review` (as reviewer and recipient)

* **Book**

  * Book details: `title`, `author`, `genre`, `book_condition`, pricing info, flags for sale/exchange, image URL, etc.
  * Linked to uploader via `user_id`.

* **Order**

  * Links buyer, seller, and book.
  * Fields like status, timestamps, etc.

* **Rating**

  * Numeric rating (e.g., 1–5)
  * Relations:

    * `ratings.belongsTo(users, { as: 'Recipient', foreignKey: 'recipient_id' })`
    * `ratings.belongsTo(users, { as: 'Rater', foreignKey: 'rater_id' })`

* **Review**

  * Text comment + reviewer/recipient relationship.

* **Genre**

  * Simple genre entity, related to books.

> For exact fields, check each model file in `models/`.

---

##  File Uploads

* Static files are served from the `/uploads` directory:

  ```js
  app.use('/uploads', express.static(path.join(__dirname, 'uploads')));
  ```
* Multer configuration lives in `middleware/upload.js`.
* Endpoints that accept files:

  * `POST /api/book/add` (field: `book_img_url`)
  * `PUT /api/user/profile/edit` (field: `profile_picture`)

The client should send these as `multipart/form-data`.

---

##  Testing

There is no dedicated test suite configured (no Jest/Mocha, etc.) in this project.

You can manually test the API using:

* [Postman](https://www.postman.com/)
* [Insomnia](https://insomnia.rest/)
* cURL or any HTTP client

---

##  Error Handling

A simple error-handling middleware is defined in `index.js`:

```js
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ message: 'Server error' });
});
```

Controllers also handle their own validation/error responses where necessary (often with appropriate HTTP status codes).

---

## Contributing

This project is mainly for **CSE334 Database Lab** coursework.

If you still want to extend it:

1. Fork the repo
2. Create a branch:
   `git checkout -b feature/my-feature`
3. Commit your changes:
   `git commit -m "Add my feature"`
4. Push to your fork:
   `git push origin feature/my-feature`
5. Open a Pull Request

---

##  License & Usage

> This project is created for educational use as part of the **CSE334 Database Lab** course.
> Not intended for production use.

---

## ✍️ Author

* **Fahad Bin Mahbub**
  GitHub: [@Fahad-Bin-Mahbub](https://github.com/Fahad-Bin-Mahbub)

```
```
